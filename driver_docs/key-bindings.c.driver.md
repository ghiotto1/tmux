# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file is responsible for managing key bindings within tmux, which are essential for defining how user inputs (key presses) are interpreted and executed as commands. The code defines a series of default key bindings and provides functions to manage these bindings, including adding, removing, and resetting them. It uses red-black trees to efficiently store and retrieve key bindings and key tables, which are collections of key bindings associated with specific contexts or modes within tmux.

The file includes several static functions for comparing and managing key bindings and key tables, such as [`key_bindings_cmp`](#key_bindings_cmp) and [`key_table_cmp`](#key_table_cmp), which are used to maintain the order of elements in the red-black trees. It also defines functions to manipulate key tables, such as [`key_bindings_get_table`](#key_bindings_get_table), [`key_bindings_add`](#key_bindings_add), and [`key_bindings_remove`](#key_bindings_remove), which allow for dynamic management of key bindings at runtime. Additionally, the file initializes default key bindings through the [`key_bindings_init`](#key_bindings_init) function, which sets up a comprehensive list of key commands that users can execute within tmux. This file is crucial for the customization and flexibility of tmux, enabling users to tailor their key bindings to suit their workflow.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### key_tables
- **Type**: `struct key_tables`
- **Description**: The `key_tables` variable is a static instance of the `key_tables` structure, which is initialized using the `RB_INITIALIZER` macro. This structure is used to manage a collection of key tables, each of which can contain key bindings for different commands or actions.
- **Use**: This variable is used to store and manage key tables, allowing for the retrieval, insertion, and manipulation of key bindings within the application.


# Functions

---
### key_table_cmp<!-- {{#callable:key_table_cmp}} -->
The `key_table_cmp` function compares two `key_table` structures based on their `name` fields.
- **Inputs**:
    - `table1`: A pointer to the first `key_table` structure to be compared.
    - `table2`: A pointer to the second `key_table` structure to be compared.
- **Control Flow**:
    - The function calls `strcmp` to compare the `name` fields of `table1` and `table2`.
- **Output**:
    - The function returns an integer value: less than, equal to, or greater than zero if `table1->name` is found, respectively, to be less than, to match, or be greater than `table2->name`.


---
### key_bindings_cmp<!-- {{#callable:key_bindings_cmp}} -->
The `key_bindings_cmp` function compares two `key_binding` structures based on their `key` values.
- **Inputs**:
    - `bd1`: A pointer to the first `key_binding` structure to be compared.
    - `bd2`: A pointer to the second `key_binding` structure to be compared.
- **Control Flow**:
    - The function first checks if the `key` of `bd1` is less than the `key` of `bd2`; if so, it returns -1.
    - If the `key` of `bd1` is greater than the `key` of `bd2`, it returns 1.
    - If neither condition is met, it returns 0, indicating the keys are equal.
- **Output**:
    - The function returns an integer: -1 if `bd1`'s key is less than `bd2`'s key, 1 if greater, and 0 if they are equal.


---
### key_bindings_free<!-- {{#callable:key_bindings_free}} -->
The `key_bindings_free` function deallocates memory associated with a `key_binding` structure, including its command list and note.
- **Inputs**:
    - `bd`: A pointer to a `key_binding` structure that needs to be freed.
- **Control Flow**:
    - Call [`cmd_list_free`](cmd.c.driver.md#cmd_list_free) to free the command list associated with the `key_binding`.
    - Free the memory allocated for the `note` field of the `key_binding`.
    - Free the memory allocated for the `key_binding` structure itself.
- **Output**:
    - The function does not return any value; it performs memory deallocation for the provided `key_binding` structure.
- **Functions called**:
    - [`cmd_list_free`](cmd.c.driver.md#cmd_list_free)


---
### key_bindings_get_table<!-- {{#callable:key_bindings_get_table}} -->
The `key_bindings_get_table` function retrieves a key table by name from a global collection, creating a new one if it doesn't exist and the `create` flag is set.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the key table to retrieve or create.
    - `create`: An integer flag indicating whether to create a new key table if one with the specified name does not exist (non-zero value means create).
- **Control Flow**:
    - Initialize a `key_table` structure `table_find` with the provided `name`.
    - Search for an existing key table in the global `key_tables` collection using `RB_FIND`.
    - If a table is found or `create` is false, return the found table or NULL.
    - If no table is found and `create` is true, allocate memory for a new `key_table`.
    - Initialize the new table's `name` with a duplicate of the provided `name`.
    - Initialize the new table's `key_bindings` and `default_key_bindings` red-black trees.
    - Set the `references` count of the new table to 1.
    - Insert the new table into the global `key_tables` collection using `RB_INSERT`.
    - Return the newly created table.
- **Output**:
    - Returns a pointer to the `key_table` structure corresponding to the specified name, or NULL if not found and not created.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### key_bindings_first_table<!-- {{#callable:key_bindings_first_table}} -->
The `key_bindings_first_table` function retrieves the first key table from a collection of key tables.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls `RB_MIN` with `key_tables` and `&key_tables` as arguments to find the minimum (first) element in the red-black tree of key tables.
    - The result of `RB_MIN` is returned, which is the first key table in the collection.
- **Output**:
    - A pointer to the first `key_table` structure in the `key_tables` red-black tree, or `NULL` if the tree is empty.


---
### key_bindings_next_table<!-- {{#callable:key_bindings_next_table}} -->
The function `key_bindings_next_table` retrieves the next key table in the red-black tree of key tables.
- **Inputs**:
    - `table`: A pointer to the current key table from which the next table is to be retrieved.
- **Control Flow**:
    - The function calls `RB_NEXT` with the `key_tables` tree and the current `table` to find the next key table in the sequence.
- **Output**:
    - Returns a pointer to the next `key_table` in the red-black tree, or `NULL` if there is no next table.


---
### key_bindings_unref_table<!-- {{#callable:key_bindings_unref_table}} -->
The `key_bindings_unref_table` function decrements the reference count of a key table and frees its resources if the count reaches zero.
- **Inputs**:
    - `table`: A pointer to a `key_table` structure whose reference count is to be decremented and potentially freed.
- **Control Flow**:
    - Decrement the reference count of the `table` by one.
    - Check if the reference count is not zero; if so, return immediately without further action.
    - Iterate over all key bindings in `table->key_bindings` using `RB_FOREACH_SAFE`, remove each binding from the tree, and free its resources using [`key_bindings_free`](#key_bindings_free).
    - Repeat the same process for `table->default_key_bindings`.
    - Free the memory allocated for `table->name`.
    - Free the memory allocated for the `table` itself.
- **Output**:
    - The function does not return a value; it performs cleanup operations on the `key_table` structure.
- **Functions called**:
    - [`key_bindings_free`](#key_bindings_free)


---
### key_bindings_get<!-- {{#callable:key_bindings_get}} -->
The `key_bindings_get` function retrieves a key binding from a specified key table using a given key code.
- **Inputs**:
    - `table`: A pointer to a `key_table` structure from which the key binding is to be retrieved.
    - `key`: A `key_code` representing the key for which the binding is to be retrieved.
- **Control Flow**:
    - A `key_binding` structure `bd` is initialized with the provided `key`.
    - The function uses `RB_FIND` to search for the key binding in the `key_bindings` red-black tree of the provided `table` using `bd`.
    - The result of the `RB_FIND` operation, which is a pointer to the found `key_binding` or `NULL` if not found, is returned.
- **Output**:
    - A pointer to the `key_binding` structure if found, or `NULL` if no binding exists for the given key in the specified table.


---
### key_bindings_get_default<!-- {{#callable:key_bindings_get_default}} -->
The function `key_bindings_get_default` retrieves a default key binding from a specified key table using a given key code.
- **Inputs**:
    - `table`: A pointer to a `key_table` structure, which contains the default key bindings.
    - `key`: A `key_code` representing the key for which the default binding is being retrieved.
- **Control Flow**:
    - A `key_binding` structure `bd` is initialized with the provided `key`.
    - The function uses `RB_FIND` to search for the `key_binding` in the `default_key_bindings` red-black tree of the provided `table`.
- **Output**:
    - Returns a pointer to the `key_binding` structure if found, or `NULL` if no default binding exists for the given key.


---
### key_bindings_first<!-- {{#callable:key_bindings_first}} -->
The `key_bindings_first` function retrieves the first key binding from a given key table.
- **Inputs**:
    - `table`: A pointer to a `key_table` structure from which the first key binding is to be retrieved.
- **Control Flow**:
    - The function calls `RB_MIN` with the `key_bindings` tree and the `key_bindings` member of the `table` structure to find the minimum (first) key binding.
- **Output**:
    - Returns a pointer to the first `key_binding` in the specified key table, or `NULL` if the table is empty.


---
### key_bindings_next<!-- {{#callable:key_bindings_next}} -->
The `key_bindings_next` function retrieves the next key binding in a red-black tree of key bindings for a given key table.
- **Inputs**:
    - `table`: A pointer to a `struct key_table`, representing the key table containing the key bindings.
    - `bd`: A pointer to a `struct key_binding`, representing the current key binding from which the next binding is to be retrieved.
- **Control Flow**:
    - The function calls `RB_NEXT`, a macro for traversing a red-black tree, to find the next key binding after the given `bd` in the `key_bindings` tree of the specified `table`.
- **Output**:
    - Returns a pointer to the next `struct key_binding` in the tree, or `NULL` if there is no next binding.


---
### key_bindings_add<!-- {{#callable:key_bindings_add}} -->
The `key_bindings_add` function adds or updates a key binding in a specified key table, associating a key with a command list and optional note, and setting a repeat flag if specified.
- **Inputs**:
    - `name`: The name of the key table where the key binding should be added or updated.
    - `key`: The key code for the key binding, with any flags masked out.
    - `note`: An optional note associated with the key binding.
    - `repeat`: An integer flag indicating whether the key binding should be repeatable.
    - `cmdlist`: A pointer to a command list to be executed when the key binding is triggered; if NULL, the function updates the note and repeat flag of an existing binding.
- **Control Flow**:
    - Retrieve or create the key table specified by `name` using [`key_bindings_get_table`](#key_bindings_get_table) with the create flag set to 1.
    - Attempt to find an existing key binding in the table for the specified `key` using [`key_bindings_get`](#key_bindings_get).
    - If `cmdlist` is NULL and a binding exists, update the note and repeat flag of the existing binding, then return.
    - If a binding exists and `cmdlist` is not NULL, remove the existing binding from the table and free its resources using [`key_bindings_free`](#key_bindings_free).
    - Allocate memory for a new key binding structure and initialize it with the provided `key`, `note`, and `cmdlist`.
    - Insert the new key binding into the table's key bindings tree using `RB_INSERT`.
    - If the `repeat` flag is set, update the binding's flags to include `KEY_BINDING_REPEAT`.
    - Print a debug log message with the key binding details using `log_debug`.
- **Output**:
    - The function does not return a value; it modifies the key table by adding or updating a key binding.
- **Functions called**:
    - [`key_bindings_get_table`](#key_bindings_get_table)
    - [`key_bindings_get`](#key_bindings_get)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`key_bindings_free`](#key_bindings_free)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`cmd_list_print`](cmd.c.driver.md#cmd_list_print)
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)


---
### key_bindings_remove<!-- {{#callable:key_bindings_remove}} -->
The `key_bindings_remove` function removes a key binding from a specified key table and cleans up the table if it becomes empty.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the key table from which the key binding should be removed.
    - `key`: A key_code representing the key for which the binding should be removed, with any flags masked out.
- **Control Flow**:
    - Retrieve the key table associated with the given name using [`key_bindings_get_table`](#key_bindings_get_table) with the create flag set to 0.
    - If the table is NULL, return immediately as there is nothing to remove.
    - Retrieve the key binding from the table using [`key_bindings_get`](#key_bindings_get) with the key masked by `~KEYC_MASK_FLAGS`.
    - If the key binding is NULL, return immediately as there is nothing to remove.
    - Log the removal of the key binding using `log_debug`.
    - Remove the key binding from the table's key bindings using `RB_REMOVE`.
    - Free the memory associated with the key binding using [`key_bindings_free`](#key_bindings_free).
    - Check if both the key bindings and default key bindings of the table are empty using `RB_EMPTY`.
    - If both are empty, remove the table from the global key tables using `RB_REMOVE` and unreference the table using [`key_bindings_unref_table`](#key_bindings_unref_table).
- **Output**:
    - The function does not return any value; it performs operations to remove a key binding and potentially a key table from the data structures.
- **Functions called**:
    - [`key_bindings_get_table`](#key_bindings_get_table)
    - [`key_bindings_get`](#key_bindings_get)
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)
    - [`key_bindings_free`](#key_bindings_free)
    - [`key_bindings_unref_table`](#key_bindings_unref_table)


---
### key_bindings_reset<!-- {{#callable:key_bindings_reset}} -->
The `key_bindings_reset` function resets a key binding to its default state in a specified key table.
- **Inputs**:
    - `name`: The name of the key table from which the key binding should be reset.
    - `key`: The key code of the binding to be reset, with any flags masked out.
- **Control Flow**:
    - Retrieve the key table associated with the given name using [`key_bindings_get_table`](#key_bindings_get_table) with the create flag set to 0.
    - If the table is not found, return immediately.
    - Retrieve the current key binding for the specified key from the table using [`key_bindings_get`](#key_bindings_get).
    - If the key binding is not found, return immediately.
    - Retrieve the default key binding for the same key using [`key_bindings_get_default`](#key_bindings_get_default).
    - If the default key binding is not found, remove the current key binding using [`key_bindings_remove`](#key_bindings_remove) and return.
    - Free the command list of the current key binding and replace it with the command list from the default key binding, incrementing the reference count.
    - Free the note of the current key binding and copy the note from the default key binding if it exists, otherwise set it to NULL.
    - Copy the flags from the default key binding to the current key binding.
- **Output**:
    - The function does not return a value; it modifies the key binding in place or removes it if no default is available.
- **Functions called**:
    - [`key_bindings_get_table`](#key_bindings_get_table)
    - [`key_bindings_get`](#key_bindings_get)
    - [`key_bindings_get_default`](#key_bindings_get_default)
    - [`key_bindings_remove`](#key_bindings_remove)
    - [`cmd_list_free`](cmd.c.driver.md#cmd_list_free)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### key_bindings_remove_table<!-- {{#callable:key_bindings_remove_table}} -->
The `key_bindings_remove_table` function removes a key table by name and updates clients using that table to use a null key table.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the key table to be removed.
- **Control Flow**:
    - Retrieve the key table with the specified name using [`key_bindings_get_table`](#key_bindings_get_table) with the create flag set to 0.
    - If the table is found, remove it from the `key_tables` red-black tree using `RB_REMOVE`.
    - Call [`key_bindings_unref_table`](#key_bindings_unref_table) to decrease the reference count and potentially free the table if no references remain.
    - Iterate over all clients using `TAILQ_FOREACH` to check if any client is using the removed key table.
    - For each client using the removed table, set their key table to NULL using [`server_client_set_key_table`](server-client.c.driver.md#server_client_set_key_table).
- **Output**:
    - The function does not return a value; it performs operations to remove a key table and update clients accordingly.
- **Functions called**:
    - [`key_bindings_get_table`](#key_bindings_get_table)
    - [`key_bindings_unref_table`](#key_bindings_unref_table)
    - [`server_client_set_key_table`](server-client.c.driver.md#server_client_set_key_table)


---
### key_bindings_reset_table<!-- {{#callable:key_bindings_reset_table}} -->
The `key_bindings_reset_table` function resets all key bindings in a specified key table to their default values or removes the table if no defaults exist.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the key table to reset.
- **Control Flow**:
    - Retrieve the key table associated with the given name using [`key_bindings_get_table`](#key_bindings_get_table) with the create flag set to 0.
    - If the table is NULL, return immediately as there is nothing to reset.
    - Check if the `default_key_bindings` tree of the table is empty using `RB_EMPTY`.
    - If the `default_key_bindings` tree is empty, call [`key_bindings_remove_table`](#key_bindings_remove_table) to remove the table and return.
    - Iterate over each key binding in the `key_bindings` tree of the table using `RB_FOREACH_SAFE`.
    - For each key binding, call [`key_bindings_reset`](#key_bindings_reset) to reset it to its default state.
- **Output**:
    - The function does not return any value; it performs operations on the key table and its bindings.
- **Functions called**:
    - [`key_bindings_get_table`](#key_bindings_get_table)
    - [`key_bindings_remove_table`](#key_bindings_remove_table)
    - [`key_bindings_reset`](#key_bindings_reset)


---
### key_bindings_init_done<!-- {{#callable:key_bindings_init_done}} -->
The `key_bindings_init_done` function duplicates all key bindings from each key table into their respective default key bindings.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which is unused in this function.
    - `data`: A void pointer to any additional data, which is also unused in this function.
- **Control Flow**:
    - Iterate over each key table in the global `key_tables` red-black tree.
    - For each key table, iterate over each key binding in the `key_bindings` red-black tree of the table.
    - Allocate memory for a new key binding structure and copy the key, note, flags, and command list from the existing key binding.
    - Increment the reference count of the command list associated with the key binding.
    - Insert the newly created key binding into the `default_key_bindings` red-black tree of the table.
    - Return `CMD_RETURN_NORMAL` to indicate successful completion.
- **Output**:
    - The function returns `CMD_RETURN_NORMAL`, indicating successful execution.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### key_bindings_init<!-- {{#callable:key_bindings_init}} -->
The `key_bindings_init` function initializes default key bindings for a terminal multiplexer by parsing and appending them to a command queue.
- **Inputs**:
    - None
- **Control Flow**:
    - A static array `defaults` is defined containing strings that represent default key bindings and their associated commands.
    - A loop iterates over each element in the `defaults` array.
    - For each element, the function `cmd_parse_from_string` is called to parse the string into a command list.
    - If parsing is unsuccessful, an error is logged and the program exits with a fatal error message.
    - If parsing is successful, the command list is appended to the command queue using [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append).
    - The command list is then freed using [`cmd_list_free`](cmd.c.driver.md#cmd_list_free).
    - After processing all default key bindings, a callback `key_bindings_init_done` is appended to the command queue.
- **Output**:
    - The function does not return any value; it initializes key bindings by appending parsed commands to a command queue.
- **Functions called**:
    - [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append)
    - [`cmdq_get_command`](cmd-queue.c.driver.md#cmdq_get_command)
    - [`cmd_list_free`](cmd.c.driver.md#cmd_list_free)


---
### key_bindings_read_only<!-- {{#callable:key_bindings_read_only}} -->
The `key_bindings_read_only` function logs an error message indicating that the client is read-only and returns an error status.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing the command queue item that triggered this function.
    - `data`: A void pointer, marked as unused, which is not utilized in this function.
- **Control Flow**:
    - The function calls `cmdq_error` with the `item` and a message indicating the client is read-only.
    - The function returns `CMD_RETURN_ERROR` to indicate an error status.
- **Output**:
    - The function returns an `enum cmd_retval` value, specifically `CMD_RETURN_ERROR`, indicating an error occurred.


---
### key_bindings_dispatch<!-- {{#callable:key_bindings_dispatch}} -->
The `key_bindings_dispatch` function processes a key binding by determining if it can be executed in a read-only context and then dispatches the appropriate command queue item.
- **Inputs**:
    - `bd`: A pointer to a `struct key_binding` representing the key binding to be dispatched.
    - `item`: A pointer to a `struct cmdq_item` representing the current command queue item, which can be used to insert the new item after it.
    - `c`: A pointer to a `struct client` representing the client context in which the key binding is being dispatched.
    - `event`: A pointer to a `struct key_event` representing the key event that triggered the key binding.
    - `fs`: A pointer to a `struct cmd_find_state` representing the command find state used to create a new command queue state.
- **Control Flow**:
    - Check if the client `c` is NULL or not read-only; if so, set `readonly` to 1.
    - If `readonly` is not set, create a new command queue item with a callback to `key_bindings_read_only`.
    - If `readonly` is set, check if the key binding has the `KEY_BINDING_REPEAT` flag and set the `CMDQ_STATE_REPEAT` flag if so.
    - Create a new command queue state using [`cmdq_new_state`](cmd-queue.c.driver.md#cmdq_new_state) with the provided `fs`, `event`, and flags.
    - Get a new command queue item using [`cmdq_get_command`](cmd-queue.c.driver.md#cmdq_get_command) with the key binding's command list and the new state.
    - Free the new command queue state using [`cmdq_free_state`](cmd-queue.c.driver.md#cmdq_free_state).
    - If `item` is not NULL, insert the new item after `item` using [`cmdq_insert_after`](cmd-queue.c.driver.md#cmdq_insert_after); otherwise, append the new item to the client `c` using [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append).
- **Output**:
    - Returns a pointer to the newly created `struct cmdq_item` that represents the dispatched command.
- **Functions called**:
    - [`cmd_list_all_have`](cmd.c.driver.md#cmd_list_all_have)
    - [`cmdq_new_state`](cmd-queue.c.driver.md#cmdq_new_state)
    - [`cmdq_get_command`](cmd-queue.c.driver.md#cmdq_get_command)
    - [`cmdq_free_state`](cmd-queue.c.driver.md#cmdq_free_state)
    - [`cmdq_insert_after`](cmd-queue.c.driver.md#cmdq_insert_after)
    - [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append)


