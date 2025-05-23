# Purpose
The provided C source code is part of the `tmux` project, specifically handling the command queue system. This code is responsible for managing the execution of commands within `tmux`, a terminal multiplexer that allows users to switch between several programs in one terminal. The command queue system is a critical component that ensures commands are processed in an orderly fashion, supporting both synchronous and asynchronous execution. The code defines structures and functions to create, manage, and execute command queue items, which can be either commands or callbacks. It also includes mechanisms for handling command states, managing client references, and processing command hooks.

Key technical components include the `cmdq_item` structure, which represents an individual command or callback in the queue, and the `cmdq_state` structure, which maintains the context for command execution, including event information and target states. The code provides functions to create new command queues, append or insert items into the queue, and execute these items while handling potential errors and hooks. The command queue system is designed to be flexible, allowing for the addition of formats and the merging of command states, which is essential for the dynamic and interactive nature of `tmux`. This file is integral to the `tmux` application, facilitating the execution of user commands and ensuring the smooth operation of the terminal multiplexer.
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
- **Description**: The `cmdq_type` is an enumeration that defines the types of items that can exist in a command queue within the tmux application. It includes two possible values: `CMDQ_COMMAND`, which indicates that the item is a command, and `CMDQ_CALLBACK`, which indicates that the item is a callback function. This enumeration is used to differentiate between different types of actions that can be queued and processed in the command queue system.


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
    - `cmdlist`: A pointer to the list of commands associated with this command queue item.
    - `cmd`: A pointer to the command associated with this command queue item.
    - `cb`: A callback function pointer for the command queue item.
    - `data`: A pointer to any additional data associated with the command queue item.
    - `entry`: A TAILQ_ENTRY structure for linking this item in a tail queue.
- **Description**: The `cmdq_item` structure represents an item in a command queue, which is part of a system for managing and executing commands in a sequential manner. Each item in the queue can represent either a command or a callback, and it contains information about the command's execution context, such as the associated client, target client, and state. The structure also includes fields for managing the item's position in the queue and for handling command execution details, such as the command list, command type, and any associated callback function. This structure is used within a larger framework to manage command execution in a controlled and organized manner, allowing for complex command sequences and interactions.


---
### cmdq_state
- **Type**: `struct`
- **Members**:
    - `references`: An integer that tracks the number of references to this state, used for memory management.
    - `flags`: An integer that holds various flags indicating the state of the command queue.
    - `formats`: A pointer to a format_tree structure, which holds additional formats for commands.
    - `event`: A key_event structure that represents the key event that triggered the command.
    - `current`: A cmd_find_state structure that represents the current default target for commands.
- **Description**: The `cmdq_state` structure is used to maintain the context for commands in a command queue. It includes information about how commands were triggered, such as the key event and flags, as well as any additional formats that may be applied to the commands. The structure also keeps track of the current default target for commands, allowing multiple commands to share the same state and potentially update the default target. The `references` field is used for managing the lifecycle of the state, ensuring it is properly freed when no longer in use.


---
### cmdq_list
- **Type**: `struct`
- **Members**:
    - `item`: A pointer to a cmdq_item structure, representing a single command queue item.
    - `list`: A cmdq_item_list structure, which is a tail queue of cmdq_item structures.
- **Description**: The `cmdq_list` structure is a data structure used to represent a command queue in the tmux codebase. It contains a pointer to a single `cmdq_item`, which represents an individual command or callback, and a `cmdq_item_list`, which is a tail queue that holds multiple `cmdq_item` structures. This structure is used to manage and process a sequence of commands or callbacks in a queue, allowing for efficient insertion, removal, and traversal of command items.


# Functions

---
### cmdq_name<!-- {{#callable:cmdq_name}} -->
The `cmdq_name` function returns a formatted string representing the name of a client or a global identifier if the client is null.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose name is to be retrieved.
- **Control Flow**:
    - Declare a static character array `s` of size 256 to hold the formatted name.
    - Check if the input client `c` is NULL; if so, return the string `"<global>"`.
    - If the client's `name` field is not NULL, format the string `s` to include the client's name enclosed in angle brackets using [`xsnprintf`](xmalloc.c.driver.md#xsnprintf).
    - If the client's `name` field is NULL, format the string `s` to include the pointer address of the client `c` enclosed in angle brackets using [`xsnprintf`](xmalloc.c.driver.md#xsnprintf).
    - Return the formatted string `s`.
- **Output**:
    - A constant character pointer to a formatted string representing the client's name or a global identifier.
- **Functions called**:
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)


---
### cmdq_get<!-- {{#callable:cmdq_get}} -->
The `cmdq_get` function retrieves the command queue associated with a given client or returns a global command queue if the client is NULL.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose command queue is to be retrieved; if NULL, the global command queue is returned.
- **Control Flow**:
    - Check if the client pointer `c` is NULL.
    - If `c` is NULL, check if the static `global_queue` is NULL.
    - If `global_queue` is NULL, initialize it by calling `cmdq_new()`.
    - Return the `global_queue` if `c` is NULL.
    - If `c` is not NULL, return the client's command queue `c->queue`.
- **Output**:
    - Returns a pointer to a `struct cmdq_list`, which is the command queue associated with the client or the global command queue if the client is NULL.
- **Functions called**:
    - [`cmdq_new`](#cmdq_new)


---
### cmdq_new<!-- {{#callable:cmdq_new}} -->
The `cmdq_new` function creates and initializes a new command queue list structure.
- **Inputs**:
    - None
- **Control Flow**:
    - Allocate memory for a new `cmdq_list` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc), which initializes the allocated memory to zero.
    - Initialize the `list` field of the `cmdq_list` structure using the `TAILQ_INIT` macro, which sets up the list head for a tail queue.
    - Return the pointer to the newly created and initialized `cmdq_list` structure.
- **Output**:
    - A pointer to a newly allocated and initialized `cmdq_list` structure.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### cmdq_free<!-- {{#callable:cmdq_free}} -->
The `cmdq_free` function deallocates a command queue structure if it is empty, otherwise it raises a fatal error.
- **Inputs**:
    - `queue`: A pointer to a `cmdq_list` structure representing the command queue to be freed.
- **Control Flow**:
    - Check if the command queue's list is not empty using `TAILQ_EMPTY` macro.
    - If the list is not empty, call `fatalx` with the message "queue not empty" to raise a fatal error.
    - If the list is empty, call `free` on the `queue` pointer to deallocate the memory.
- **Output**:
    - The function does not return any value; it either frees the memory of the queue or raises a fatal error if the queue is not empty.


---
### cmdq_get_name<!-- {{#callable:cmdq_get_name}} -->
The function `cmdq_get_name` retrieves the name of a command queue item.
- **Inputs**:
    - `item`: A pointer to a `struct cmdq_item`, which represents an item in the command queue.
- **Control Flow**:
    - The function directly returns the `name` field of the `cmdq_item` structure pointed to by `item`.
- **Output**:
    - The function returns a `const char *`, which is the name of the command queue item.


---
### cmdq_get_client<!-- {{#callable:cmdq_get_client}} -->
The `cmdq_get_client` function retrieves the client associated with a given command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure from which the client is to be retrieved.
- **Control Flow**:
    - The function directly returns the `client` member of the `cmdq_item` structure pointed to by `item`.
- **Output**:
    - A pointer to a `client` structure, which is the client associated with the given command queue item.


---
### cmdq_get_target_client<!-- {{#callable:cmdq_get_target_client}} -->
The `cmdq_get_target_client` function retrieves the target client associated with a given command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which represents an item in the command queue.
- **Control Flow**:
    - The function takes a single argument, `item`, which is a pointer to a `cmdq_item` structure.
    - It directly accesses the `target_client` member of the `cmdq_item` structure pointed to by `item`.
    - The function returns the value of the `target_client` member.
- **Output**:
    - A pointer to a `client` structure, representing the target client associated with the given command queue item.


---
### cmdq_get_state<!-- {{#callable:cmdq_get_state}} -->
The `cmdq_get_state` function retrieves the state associated with a given command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure from which the state is to be retrieved.
- **Control Flow**:
    - The function takes a single argument, `item`, which is a pointer to a `cmdq_item` structure.
    - It directly returns the `state` member of the `cmdq_item` structure pointed to by `item`.
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
    - A pointer to a `cmd_find_state` structure representing the source command find state of the given command queue item.


---
### cmdq_get_event<!-- {{#callable:cmdq_get_event}} -->
The `cmdq_get_event` function retrieves the `key_event` structure from the state of a given command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which represents an item in the command queue.
- **Control Flow**:
    - The function accesses the `state` member of the `cmdq_item` structure pointed to by `item`.
    - It then returns the address of the `event` member within the `cmdq_state` structure.
- **Output**:
    - A pointer to the `key_event` structure associated with the command queue item's state.


---
### cmdq_get_current<!-- {{#callable:cmdq_get_current}} -->
The `cmdq_get_current` function retrieves the current command find state from a given command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure from which the current command find state is to be retrieved.
- **Control Flow**:
    - The function accesses the `state` member of the `cmdq_item` structure pointed to by `item`.
    - It then returns the address of the `current` member of the `cmdq_state` structure, which is pointed to by the `state` member.
- **Output**:
    - A pointer to the `cmd_find_state` structure representing the current command find state of the given command queue item.


---
### cmdq_get_flags<!-- {{#callable:cmdq_get_flags}} -->
The `cmdq_get_flags` function retrieves the flags from the state of a given command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which represents an item in the command queue.
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
    - Allocate memory for a new `cmdq_state` structure and initialize its `references` to 1 and `flags` to the provided `flags` argument.
    - If the `event` argument is not NULL, copy its contents into the `event` field of the new state; otherwise, set the `event.key` to `KEYC_NONE`.
    - If the `current` argument is not NULL and represents a valid state, copy it into the `current` field of the new state using [`cmd_find_copy_state`](cmd-find.c.driver.md#cmd_find_copy_state); otherwise, clear the `current` field using [`cmd_find_clear_state`](cmd-find.c.driver.md#cmd_find_clear_state).
    - Return the newly created and initialized `cmdq_state` structure.
- **Output**:
    - A pointer to the newly created `cmdq_state` structure.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`cmd_find_valid_state`](cmd-find.c.driver.md#cmd_find_valid_state)
    - [`cmd_find_copy_state`](cmd-find.c.driver.md#cmd_find_copy_state)
    - [`cmd_find_clear_state`](cmd-find.c.driver.md#cmd_find_clear_state)


---
### cmdq_link_state<!-- {{#callable:cmdq_link_state}} -->
The `cmdq_link_state` function increments the reference count of a given `cmdq_state` structure and returns the updated state.
- **Inputs**:
    - `state`: A pointer to a `cmdq_state` structure whose reference count is to be incremented.
- **Control Flow**:
    - The function takes a `cmdq_state` pointer as input.
    - It increments the `references` field of the `cmdq_state` structure by one.
    - The function then returns the updated `cmdq_state` pointer.
- **Output**:
    - The function returns the same `cmdq_state` pointer that was passed in, with its `references` field incremented.


---
### cmdq_copy_state<!-- {{#callable:cmdq_copy_state}} -->
The `cmdq_copy_state` function creates a new command queue state by copying the current state and optionally replacing the current target with a provided one.
- **Inputs**:
    - `state`: A pointer to a `cmdq_state` structure representing the current command queue state to be copied.
    - `current`: A pointer to a `cmd_find_state` structure representing the current target state to be used in the new state, or NULL to use the current target from the provided state.
- **Control Flow**:
    - Check if the `current` argument is not NULL.
    - If `current` is not NULL, call [`cmdq_new_state`](#cmdq_new_state) with `current`, the event from `state`, and the flags from `state` to create a new state.
    - If `current` is NULL, call [`cmdq_new_state`](#cmdq_new_state) with the current target from `state`, the event from `state`, and the flags from `state` to create a new state.
- **Output**:
    - Returns a pointer to a newly created `cmdq_state` structure that is a copy of the provided state, with the current target replaced if specified.
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
    - If the `formats` field of the `state` is not NULL, call [`format_free`](format.c.driver.md#format_free) to free the formats.
    - Free the memory allocated for the `state` structure.
- **Output**:
    - The function does not return a value; it performs memory management by freeing the `cmdq_state` structure if its reference count is zero.
- **Functions called**:
    - [`format_free`](format.c.driver.md#format_free)


---
### cmdq_add_format<!-- {{#callable:cmdq_add_format}} -->
The `cmdq_add_format` function adds a formatted string to the command queue state's format tree using a specified key.
- **Inputs**:
    - `state`: A pointer to a `cmdq_state` structure, representing the current state of the command queue.
    - `key`: A constant character pointer representing the key under which the formatted string will be stored.
    - `fmt`: A constant character pointer representing the format string, similar to printf-style formatting.
    - `...`: A variable argument list that provides the values to be formatted according to the format string.
- **Control Flow**:
    - Initialize a variable argument list `ap` using `va_start` with `fmt` as the last fixed argument.
    - Use [`xvasprintf`](xmalloc.c.driver.md#xvasprintf) to allocate and format a string `value` using the format string `fmt` and the variable arguments `ap`.
    - End the variable argument list using `va_end`.
    - Check if `state->formats` is `NULL`; if so, initialize it using [`format_create`](format.c.driver.md#format_create) with default parameters.
    - Add the formatted string `value` to `state->formats` using `format_add` with the specified `key`.
    - Free the allocated memory for `value` using `free`.
- **Output**:
    - The function does not return a value; it modifies the `formats` field of the `cmdq_state` structure by adding a new formatted string.
- **Functions called**:
    - [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)
    - [`format_create`](format.c.driver.md#format_create)


---
### cmdq_add_formats<!-- {{#callable:cmdq_add_formats}} -->
The `cmdq_add_formats` function adds a set of formats from a `format_tree` to a `cmdq_state`, initializing the state's formats if they are not already set.
- **Inputs**:
    - `state`: A pointer to a `cmdq_state` structure, which represents the context for commands on the command queue, including any additional formats.
    - `ft`: A pointer to a `format_tree` structure, which contains the formats to be added to the command queue state.
- **Control Flow**:
    - Check if the `formats` field of the `state` is `NULL`.
    - If `formats` is `NULL`, initialize it using [`format_create`](format.c.driver.md#format_create) with default parameters.
    - Merge the formats from the `format_tree` `ft` into the `state->formats` using [`format_merge`](format.c.driver.md#format_merge).
- **Output**:
    - The function does not return a value; it modifies the `formats` field of the `cmdq_state` structure in place.
- **Functions called**:
    - [`format_create`](format.c.driver.md#format_create)
    - [`format_merge`](format.c.driver.md#format_merge)


---
### cmdq_merge_formats<!-- {{#callable:cmdq_merge_formats}} -->
The `cmdq_merge_formats` function merges command-related formats from a command queue item into a given format tree.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing a command queue item that may contain a command and associated formats.
    - `ft`: A pointer to a `format_tree` structure, which is the target format tree where formats from the command queue item will be merged.
- **Control Flow**:
    - Check if the `cmd` field of the `item` is not NULL.
    - If not NULL, retrieve the command entry using [`cmd_get_entry`](cmd.c.driver.md#cmd_get_entry) and add the command name to the format tree `ft` with the key 'command'.
    - Check if the `formats` field of the `item`'s state is not NULL.
    - If not NULL, merge the formats from the `item`'s state into the format tree `ft`.
- **Output**:
    - The function does not return a value; it modifies the `format_tree` pointed to by `ft` by adding or merging formats from the `cmdq_item`.
- **Functions called**:
    - [`cmd_get_entry`](cmd.c.driver.md#cmd_get_entry)
    - [`format_merge`](format.c.driver.md#format_merge)


---
### cmdq_append<!-- {{#callable:cmdq_append}} -->
The `cmdq_append` function appends a command queue item to a client's command queue, handling client references and logging.
- **Inputs**:
    - `c`: A pointer to a `client` structure, representing the client to which the command queue item is to be appended.
    - `item`: A pointer to a `cmdq_item` structure, representing the command queue item to be appended.
- **Control Flow**:
    - Retrieve the command queue associated with the client `c` using [`cmdq_get`](#cmdq_get) function.
    - Enter a loop that continues until `item` is NULL.
    - In each iteration, store the next item in the sequence in `next` and set `item->next` to NULL.
    - If the client `c` is not NULL, increment its reference count.
    - Assign the client `c` to `item->client`.
    - Assign the retrieved queue to `item->queue`.
    - Insert the `item` at the tail of the queue's list using `TAILQ_INSERT_TAIL`.
    - Log a debug message with the function name, client queue name, and item name.
    - Move to the next item in the sequence by setting `item` to `next`.
    - Return the last item in the queue's list using `TAILQ_LAST`.
- **Output**:
    - Returns a pointer to the last `cmdq_item` in the queue's list after appending the new item(s).
- **Functions called**:
    - [`cmdq_get`](#cmdq_get)
    - [`cmdq_name`](#cmdq_name)


---
### cmdq_insert_after<!-- {{#callable:cmdq_insert_after}} -->
The `cmdq_insert_after` function inserts a command queue item or a series of items into a command queue immediately after a specified item.
- **Inputs**:
    - `after`: A pointer to the `cmdq_item` structure after which the new item(s) should be inserted.
    - `item`: A pointer to the `cmdq_item` structure that is to be inserted after the `after` item.
- **Control Flow**:
    - Retrieve the client and queue associated with the `after` item.
    - Enter a loop that continues as long as `item` is not NULL.
    - Within the loop, store the next item in the sequence in `next`.
    - Set the `next` pointer of the current `item` to the `next` pointer of `after`.
    - Update the `next` pointer of `after` to point to the current `item`.
    - If the client `c` is not NULL, increment its reference count.
    - Assign the client `c` and queue to the current `item`.
    - Insert the current `item` into the queue list after the `after` item using `TAILQ_INSERT_AFTER`.
    - Log a debug message indicating the insertion of the item.
    - Update `after` to be the current `item` and move to the next item in the sequence by setting `item` to `next`.
    - Exit the loop when `item` becomes NULL.
- **Output**:
    - Returns a pointer to the last `cmdq_item` that was inserted.
- **Functions called**:
    - [`cmdq_name`](#cmdq_name)


---
### cmdq_insert_hook<!-- {{#callable:cmdq_insert_hook}} -->
The `cmdq_insert_hook` function inserts a hook command into the command queue for a given session, based on a specified format and arguments.
- **Inputs**:
    - `s`: A pointer to a `struct session`, representing the session in which the hook is to be inserted; if NULL, global session options are used.
    - `item`: A pointer to a `struct cmdq_item`, representing the command queue item where the hook will be inserted.
    - `current`: A pointer to a `struct cmd_find_state`, representing the current command find state.
    - `fmt`: A format string used to generate the hook name, followed by additional arguments for the format.
- **Control Flow**:
    - Check if the current command queue state has the `CMDQ_STATE_NOHOOKS` flag set; if so, return immediately without inserting the hook.
    - Determine the options to use based on whether the session `s` is NULL or not.
    - Use `va_start` and `va_end` to handle the variable arguments and format the hook name using [`xvasprintf`](xmalloc.c.driver.md#xvasprintf).
    - Retrieve the options entry for the formatted hook name; if it doesn't exist, free the name and return.
    - Log the hook execution and create a new command queue state with `CMDQ_STATE_NOHOOKS` to prevent updates to the current target or formats.
    - Add the hook name and arguments to the new state's format tree.
    - Iterate over the arguments and flags of the command, adding each to the new state's format tree with specific keys.
    - Iterate over the options array items for the hook, retrieving the command list and inserting each command into the queue after the current item.
    - Free the new command queue state and the formatted hook name.
- **Output**:
    - The function does not return a value; it modifies the command queue by inserting hook commands based on the specified format and arguments.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)
    - [`options_get`](options.c.driver.md#options_get)
    - [`cmdq_new_state`](#cmdq_new_state)
    - [`cmdq_add_format`](#cmdq_add_format)
    - [`args_print`](arguments.c.driver.md#args_print)
    - [`args_count`](arguments.c.driver.md#args_count)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`args_string`](arguments.c.driver.md#args_string)
    - [`args_first`](arguments.c.driver.md#args_first)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`args_first_value`](arguments.c.driver.md#args_first_value)
    - [`args_next_value`](arguments.c.driver.md#args_next_value)
    - [`args_next`](arguments.c.driver.md#args_next)
    - [`options_array_first`](options.c.driver.md#options_array_first)
    - [`options_array_item_value`](options.c.driver.md#options_array_item_value)
    - [`cmdq_get_command`](#cmdq_get_command)
    - [`cmdq_insert_after`](#cmdq_insert_after)
    - [`cmdq_append`](#cmdq_append)
    - [`options_array_next`](options.c.driver.md#options_array_next)
    - [`cmdq_free_state`](#cmdq_free_state)


---
### cmdq_continue<!-- {{#callable:cmdq_continue}} -->
The `cmdq_continue` function clears the CMDQ_WAITING flag from a given command queue item, allowing it to continue processing.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing an item in the command queue whose CMDQ_WAITING flag needs to be cleared.
- **Control Flow**:
    - The function accesses the `flags` field of the `cmdq_item` structure pointed to by `item`.
    - It uses a bitwise AND operation with the negation of `CMDQ_WAITING` to clear the CMDQ_WAITING flag from the `flags` field.
- **Output**:
    - The function does not return any value; it modifies the `flags` field of the `cmdq_item` structure in place.


---
### cmdq_remove<!-- {{#callable:cmdq_remove}} -->
The `cmdq_remove` function removes a command queue item from its queue and frees associated resources.
- **Inputs**:
    - `item`: A pointer to the `cmdq_item` structure that represents the command queue item to be removed.
- **Control Flow**:
    - Check if the `client` field of the `item` is not NULL, and if so, call [`server_client_unref`](server-client.c.driver.md#server_client_unref) to decrease the reference count of the client.
    - Check if the `cmdlist` field of the `item` is not NULL, and if so, call [`cmd_list_free`](cmd.c.driver.md#cmd_list_free) to free the command list associated with the item.
    - Call [`cmdq_free_state`](#cmdq_free_state) to free the state associated with the item.
    - Remove the item from its queue using `TAILQ_REMOVE`.
    - Free the memory allocated for the `name` field of the item.
    - Free the memory allocated for the `item` itself.
- **Output**:
    - The function does not return a value; it performs cleanup and deallocation of resources associated with the command queue item.
- **Functions called**:
    - [`server_client_unref`](server-client.c.driver.md#server_client_unref)
    - [`cmd_list_free`](cmd.c.driver.md#cmd_list_free)
    - [`cmdq_free_state`](#cmdq_free_state)


---
### cmdq_remove_group<!-- {{#callable:cmdq_remove_group}} -->
The `cmdq_remove_group` function removes all subsequent command queue items that belong to the same group as the specified item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing the command queue item whose group is to be removed from the queue.
- **Control Flow**:
    - Check if the `group` attribute of the `item` is zero; if so, return immediately as there is no group to remove.
    - Initialize `this` to the next item in the queue after `item` using `TAILQ_NEXT`.
    - Iterate over the queue starting from `this`, checking each item's `group` attribute.
    - If an item's `group` matches the `group` of the input `item`, call [`cmdq_remove`](#cmdq_remove) to remove it from the queue.
    - Continue iterating through the queue until all items have been checked.
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
    - The function takes two parameters, `item` and `data`, both of which are marked as unused, indicating they are not utilized within the function body.
    - The function immediately returns the constant `CMD_RETURN_NORMAL`, indicating a normal command execution result.
- **Output**:
    - The function returns an `enum cmd_retval` value, specifically `CMD_RETURN_NORMAL`, which signifies a normal, successful command execution.


---
### cmdq_get_command<!-- {{#callable:cmdq_get_command}} -->
The `cmdq_get_command` function creates a linked list of command queue items from a given command list and state, initializing each item with relevant command and state information.
- **Inputs**:
    - `cmdlist`: A pointer to a `struct cmd_list` which contains the list of commands to be processed.
    - `state`: A pointer to a `struct cmdq_state` which represents the current state of the command queue; if NULL, a new state is created.
- **Control Flow**:
    - Check if the first command in `cmdlist` is NULL; if so, return a callback item for an empty command.
    - If `state` is NULL, create a new state and set a flag indicating it was created.
    - Iterate over each command in `cmdlist` using a while loop.
    - For each command, retrieve its entry and allocate memory for a new `cmdq_item`.
    - Initialize the `cmdq_item` with the command's entry name, type, group, and link it to the current state.
    - Increment the reference count of `cmdlist`.
    - Log debug information about the created item.
    - Link the newly created item to the list of items, maintaining the order.
    - If a new state was created, free it after processing all commands.
    - Return the first item in the linked list of command queue items.
- **Output**:
    - Returns a pointer to the first `cmdq_item` in the linked list of command queue items created from the `cmdlist`.
- **Functions called**:
    - [`cmd_list_first`](cmd.c.driver.md#cmd_list_first)
    - [`cmdq_new_state`](#cmdq_new_state)
    - [`cmd_get_entry`](cmd.c.driver.md#cmd_get_entry)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`cmd_get_group`](cmd.c.driver.md#cmd_get_group)
    - [`cmdq_link_state`](#cmdq_link_state)
    - [`cmd_list_next`](cmd.c.driver.md#cmd_list_next)
    - [`cmdq_free_state`](#cmdq_free_state)


---
### cmdq_find_flag<!-- {{#callable:cmdq_find_flag}} -->
The `cmdq_find_flag` function determines the target state for a command queue item based on a specified flag and updates the find state accordingly.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item being processed.
    - `fs`: A pointer to a `cmd_find_state` structure where the target state will be stored.
    - `flag`: A pointer to a `cmd_entry_flag` structure that specifies the flag to be used for finding the target.
- **Control Flow**:
    - Check if the flag's value is zero; if so, call [`cmd_find_from_client`](cmd-find.c.driver.md#cmd_find_from_client) to set the find state from the target client and return `CMD_RETURN_NORMAL`.
    - Retrieve the flag's value using [`args_get`](arguments.c.driver.md#args_get) and [`cmd_get_args`](cmd.c.driver.md#cmd_get_args).
    - Call [`cmd_find_target`](cmd-find.c.driver.md#cmd_find_target) with the retrieved value and other parameters to find the target.
    - If [`cmd_find_target`](cmd-find.c.driver.md#cmd_find_target) returns non-zero, clear the find state using [`cmd_find_clear_state`](cmd-find.c.driver.md#cmd_find_clear_state) and return `CMD_RETURN_ERROR`.
    - If [`cmd_find_target`](cmd-find.c.driver.md#cmd_find_target) is successful, return `CMD_RETURN_NORMAL`.
- **Output**:
    - The function returns an `enum cmd_retval`, which is either `CMD_RETURN_NORMAL` if the target was found successfully or `CMD_RETURN_ERROR` if there was an error finding the target.
- **Functions called**:
    - [`cmd_find_from_client`](cmd-find.c.driver.md#cmd_find_from_client)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmd_find_target`](cmd-find.c.driver.md#cmd_find_target)
    - [`cmd_find_clear_state`](cmd-find.c.driver.md#cmd_find_clear_state)


---
### cmdq_add_message<!-- {{#callable:cmdq_add_message}} -->
The `cmdq_add_message` function logs a message about a command execution, including user and key information if available, to the server.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item for which the message is being added.
- **Control Flow**:
    - Retrieve the client and state from the `cmdq_item` structure.
    - Generate a string representation of the command using [`cmd_print`](cmd.c.driver.md#cmd_print).
    - If a client is associated with the item, retrieve the UID of the peer client and compare it with the current user's UID.
    - If the UIDs differ, attempt to get the username associated with the peer UID; if successful, format it as '[username]', otherwise use '[unknown]'.
    - If the client has an associated session and a key event is present, look up the key string and log a message including the client name, user, key, and command.
    - If no key event is present, log a message including the client name, user, and command.
    - If no client is associated with the item, log a message with just the command.
    - Free any dynamically allocated memory for the user and command strings.
- **Output**:
    - The function does not return a value; it logs a message to the server.
- **Functions called**:
    - [`cmd_print`](cmd.c.driver.md#cmd_print)
    - [`proc_get_peer_uid`](proc.c.driver.md#proc_get_peer_uid)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)


---
### cmdq_fire_command<!-- {{#callable:cmdq_fire_command}} -->
The `cmdq_fire_command` function executes a command from a command queue item, handling client and target client resolution, command execution, and error or hook handling.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item to be executed.
- **Control Flow**:
    - Retrieve the command name and associated state, command, arguments, and entry from the `cmdq_item`.
    - If configuration is finished, add a message for the command; log the command if the log level is greater than 1.
    - Determine if the command is a control command and set the appropriate flags.
    - Resolve the client for the command if it is not already set.
    - Check command entry flags to determine if a client can fail or if specific client flags ('c' or 't') are required, and resolve the target client accordingly.
    - Find and set the source and target flags for the command; if any flag resolution fails, return an error.
    - Execute the command using the entry's `exec` function; if execution fails, return an error.
    - If the command has an after-hook flag, determine the appropriate state and insert the after-hook.
    - Restore the original client and handle any errors by inserting a command-error hook and guarding the command with an error guard; otherwise, guard the command with an end guard.
    - Return the command execution result.
- **Output**:
    - Returns an `enum cmd_retval` indicating the result of the command execution, which can be normal, error, or wait.
- **Functions called**:
    - [`cmdq_name`](#cmdq_name)
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmd_get_entry`](cmd.c.driver.md#cmd_get_entry)
    - [`cmdq_add_message`](#cmdq_add_message)
    - [`log_get_level`](log.c.driver.md#log_get_level)
    - [`cmd_print`](cmd.c.driver.md#cmd_print)
    - [`cmdq_guard`](#cmdq_guard)
    - [`cmd_find_client`](cmd-find.c.driver.md#cmd_find_client)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`cmdq_find_flag`](#cmdq_find_flag)
    - [`cmd_find_valid_state`](cmd-find.c.driver.md#cmd_find_valid_state)
    - [`cmd_find_from_client`](cmd-find.c.driver.md#cmd_find_from_client)
    - [`cmdq_insert_hook`](#cmdq_insert_hook)


---
### cmdq_get_callback1<!-- {{#callable:cmdq_get_callback1}} -->
The `cmdq_get_callback1` function creates and initializes a new command queue item for a callback with a specified name, callback function, and associated data.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the callback item.
    - `cb`: A function pointer of type `cmdq_cb` representing the callback function to be associated with the command queue item.
    - `data`: A void pointer to any data that should be associated with the callback item.
- **Control Flow**:
    - Allocate memory for a new `cmdq_item` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc) and assign it to the `item` variable.
    - Format the `name` and address of the `item` into a string and assign it to `item->name` using [`xasprintf`](xmalloc.c.driver.md#xasprintf).
    - Set the `item->type` to `CMDQ_CALLBACK` to indicate that this item is a callback type.
    - Initialize `item->group` to 0, indicating no specific group association.
    - Create a new command queue state with [`cmdq_new_state`](#cmdq_new_state) and assign it to `item->state`.
    - Assign the provided callback function `cb` to `item->cb`.
    - Assign the provided data pointer `data` to `item->data`.
    - Return the initialized `cmdq_item` structure.
- **Output**:
    - Returns a pointer to a newly created and initialized `cmdq_item` structure representing a callback item in the command queue.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`cmdq_new_state`](#cmdq_new_state)


---
### cmdq_error_callback<!-- {{#callable:cmdq_error_callback}} -->
The `cmdq_error_callback` function logs an error message associated with a command queue item and then frees the allocated memory for the error message.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing the command queue item associated with the error.
    - `data`: A pointer to a character string containing the error message to be logged.
- **Control Flow**:
    - The function casts the `data` pointer to a `char*` and assigns it to the `error` variable.
    - It calls the [`cmdq_error`](#cmdq_error) function, passing the `item` and the `error` message to log the error.
    - The function then frees the memory allocated for the `error` message using `free(error)`.
    - Finally, it returns `CMD_RETURN_NORMAL` to indicate normal completion.
- **Output**:
    - The function returns `CMD_RETURN_NORMAL`, indicating that the callback completed without any issues.
- **Functions called**:
    - [`cmdq_error`](#cmdq_error)


---
### cmdq_get_error<!-- {{#callable:cmdq_get_error}} -->
The `cmdq_get_error` function creates a command queue item that represents an error callback with a specified error message.
- **Inputs**:
    - `error`: A constant character pointer representing the error message to be used in the error callback.
- **Control Flow**:
    - The function duplicates the input error string using [`xstrdup`](xmalloc.c.driver.md#xstrdup) to ensure it is safely stored.
    - It calls `cmdq_get_callback` with `cmdq_error_callback` as the callback function and the duplicated error string as data.
    - The `cmdq_get_callback` function returns a new `cmdq_item` configured to execute the error callback with the provided error message.
- **Output**:
    - The function returns a pointer to a `cmdq_item` structure, which is configured to execute an error callback with the specified error message.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### cmdq_fire_callback<!-- {{#callable:cmdq_fire_callback}} -->
The `cmdq_fire_callback` function executes a callback function associated with a command queue item, passing the item and its data as arguments.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which contains the callback function and data to be executed.
- **Control Flow**:
    - The function directly calls the callback function (`cb`) stored in the `cmdq_item` structure, passing the `item` and its `data` as arguments.
    - The result of the callback function is returned as the result of `cmdq_fire_callback`.
- **Output**:
    - The function returns an `enum cmd_retval`, which is the result of executing the callback function associated with the `cmdq_item`.


---
### cmdq_next<!-- {{#callable:cmdq_next}} -->
The `cmdq_next` function processes the next item in a command queue for a given client, executing commands or callbacks and handling their results.
- **Inputs**:
    - `c`: A pointer to a `struct client`, representing the client whose command queue is to be processed.
- **Control Flow**:
    - Retrieve the command queue associated with the client using [`cmdq_get`](#cmdq_get) and get the queue's name using [`cmdq_name`](#cmdq_name).
    - Check if the queue is empty or if the first item is waiting; if so, log the status and return 0.
    - Enter a loop to process each item in the queue until no more items are left.
    - For each item, check if it is waiting; if so, log the status and exit the loop.
    - If the item has not been fired, set its time and number, then execute it based on its type (command or callback).
    - If a command returns an error, remove subsequent commands in the same group using [`cmdq_remove_group`](#cmdq_remove_group).
    - If a command returns a wait status, set the waiting flag and exit the loop.
    - Remove the processed item from the queue using [`cmdq_remove`](#cmdq_remove).
    - Log the exit status and return the number of processed items.
- **Output**:
    - Returns the number of items processed from the command queue as an unsigned integer.
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
    - `c`: A pointer to a `client` structure representing the client for which the command queue is being queried.
- **Control Flow**:
    - Retrieve the command queue associated with the client `c` using `cmdq_get(c)`.
    - Check if the current item in the queue is `NULL`; if so, return `NULL`.
    - Check if the current item's flags include `CMDQ_WAITING`; if so, return `NULL`.
    - Return the current item in the queue.
- **Output**:
    - A pointer to the currently running `cmdq_item` for the client, or `NULL` if there is no running item or if the item is waiting.
- **Functions called**:
    - [`cmdq_get`](#cmdq_get)


---
### cmdq_guard<!-- {{#callable:cmdq_guard}} -->
The `cmdq_guard` function writes a formatted control message to a client if the client is in control mode.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which contains information about the command queue item, including the client, time, and number.
    - `guard`: A constant character pointer representing the guard string to be included in the control message.
    - `flags`: An integer representing additional flags to be included in the control message.
- **Control Flow**:
    - Retrieve the client (`c`), time (`t`), and number from the `cmdq_item` structure pointed to by `item`.
    - Check if the client (`c`) is not NULL and if the client is in control mode (`c->flags & CLIENT_CONTROL`).
    - If both conditions are true, call `control_write` to send a formatted message to the client, including the guard string, time, number, and flags.
- **Output**:
    - The function does not return a value; it performs an action by potentially writing a message to a client.


---
### cmdq_print_data<!-- {{#callable:cmdq_print_data}} -->
The `cmdq_print_data` function sends a message to a client by printing the contents of an `evbuffer` to the client's output.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which contains information about the command queue item, including the client to which the message should be sent.
    - `evb`: A pointer to an `evbuffer` structure, which holds the data to be printed to the client's output.
- **Control Flow**:
    - The function calls [`server_client_print`](server-client.c.driver.md#server_client_print), passing the client from the `cmdq_item`, a flag set to 1, and the `evbuffer` to print the data to the client.
- **Output**:
    - The function does not return any value; it performs an action by printing data to a client's output.
- **Functions called**:
    - [`server_client_print`](server-client.c.driver.md#server_client_print)


---
### cmdq_print<!-- {{#callable:cmdq_print}} -->
The `cmdq_print` function formats a message using a variable argument list and sends it to be printed by the command queue system.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing the command queue item that will receive the printed message.
    - `fmt`: A format string similar to those used in `printf`, specifying how the subsequent arguments should be formatted.
    - `...`: A variable number of arguments that will be formatted according to the `fmt` string.
- **Control Flow**:
    - Create a new `evbuffer` object to hold the formatted message.
    - Check if the `evbuffer` was successfully created; if not, terminate the program with an out-of-memory error.
    - Initialize a `va_list` to handle the variable arguments and format them into the `evbuffer` using `evbuffer_add_vprintf`.
    - End the use of the `va_list`.
    - Call [`cmdq_print_data`](#cmdq_print_data) to send the formatted message in the `evbuffer` to the specified `cmdq_item`.
    - Free the `evbuffer` to release the allocated memory.
- **Output**:
    - The function does not return a value; it outputs the formatted message to the command queue system associated with the provided `cmdq_item`.
- **Functions called**:
    - [`cmdq_print_data`](#cmdq_print_data)


---
### cmdq_error<!-- {{#callable:cmdq_error}} -->
The `cmdq_error` function logs and handles error messages for command queue items, adjusting the output based on the client and session state.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item associated with the error.
    - `fmt`: A format string for the error message, similar to `printf`.
    - `...`: Additional arguments for the format string.
- **Control Flow**:
    - Initialize a variable argument list and format the error message using [`xvasprintf`](xmalloc.c.driver.md#xvasprintf).
    - Log the error message using `log_debug`.
    - Check if the client (`c`) associated with the item is `NULL`.
    - If the client is `NULL`, retrieve the source file and line number of the command and add the error message to the configuration causes.
    - If the client is not `NULL` and either the session is `NULL` or the client has the `CLIENT_CONTROL` flag, add the message to the server and handle UTF-8 sanitization if necessary.
    - If the client has the `CLIENT_CONTROL` flag, write the message to the control interface; otherwise, write it to the file error interface and set the client's return value to 1.
    - If the client has a session, capitalize the first letter of the message and set it as a status message for the client.
    - Free the allocated memory for the error message.
- **Output**:
    - The function does not return a value; it handles error logging and message output based on the client and session state.
- **Functions called**:
    - [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)
    - [`cmd_get_source`](cmd.c.driver.md#cmd_get_source)
    - [`utf8_sanitize`](utf8.c.driver.md#utf8_sanitize)


