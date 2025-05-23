# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file is responsible for managing key bindings within tmux, providing functionality to define, retrieve, modify, and remove key bindings associated with different key tables. It includes functions to compare key bindings and key tables, manage the lifecycle of key tables, and handle the addition and removal of key bindings. The file also defines default key bindings for various actions, such as window and pane management, and includes logic to handle key binding dispatching, ensuring that commands are executed in the appropriate context.

The code utilizes red-black trees to efficiently store and manage key bindings and key tables, ensuring quick access and modification. It defines several static functions for internal operations, such as comparing and freeing key bindings, and provides public functions to interact with key tables and bindings. The file also includes initialization routines to set up default key bindings and a mechanism to handle read-only client states. This code is integral to the tmux application, as it defines the user interface's responsiveness to keyboard and mouse inputs, allowing users to customize their interaction with tmux sessions.
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
- **Description**: The `key_tables` variable is a static instance of the `struct key_tables` data structure, which is initialized using the `RB_INITIALIZER` macro. This structure is part of a red-black tree implementation used to manage key tables in the application.
- **Use**: The `key_tables` variable is used to store and manage key tables, allowing for efficient insertion, deletion, and lookup operations on key bindings within the application.


# Functions

---
### key_table_cmp<!-- {{#callable:key_table_cmp}} -->
The `key_table_cmp` function compares two `key_table` structures based on their `name` fields using the `strcmp` function.
- **Inputs**:
    - `table1`: A pointer to the first `key_table` structure to be compared.
    - `table2`: A pointer to the second `key_table` structure to be compared.
- **Control Flow**:
    - The function takes two `key_table` pointers as arguments.
    - It calls the `strcmp` function to compare the `name` fields of the two `key_table` structures.
    - The result of the `strcmp` function is returned as the output of `key_table_cmp`.
- **Output**:
    - An integer value indicating the result of the comparison: a negative value if `table1->name` is less than `table2->name`, zero if they are equal, and a positive value if `table1->name` is greater than `table2->name`.


---
### key_bindings_cmp<!-- {{#callable:key_bindings_cmp}} -->
The `key_bindings_cmp` function compares two `key_binding` structures based on their `key` values and returns an integer indicating their relative order.
- **Inputs**:
    - `bd1`: A pointer to the first `key_binding` structure to be compared.
    - `bd2`: A pointer to the second `key_binding` structure to be compared.
- **Control Flow**:
    - Check if the `key` of `bd1` is less than the `key` of `bd2`; if true, return -1.
    - Check if the `key` of `bd1` is greater than the `key` of `bd2`; if true, return 1.
    - If neither condition is met, return 0, indicating the keys are equal.
- **Output**:
    - An integer: -1 if `bd1`'s key is less than `bd2`'s key, 1 if greater, and 0 if they are equal.


---
### key_bindings_free<!-- {{#callable:key_bindings_free}} -->
The `key_bindings_free` function deallocates memory associated with a `key_binding` structure, including its command list and note.
- **Inputs**:
    - `bd`: A pointer to a `key_binding` structure that needs to be freed.
- **Control Flow**:
    - Call `cmd_list_free` to free the command list associated with the `key_binding`.
    - Free the memory allocated for the `note` field of the `key_binding`.
    - Free the memory allocated for the `key_binding` structure itself.
- **Output**:
    - The function does not return any value; it performs memory deallocation for the given `key_binding` structure.


---
### key_bindings_get_table<!-- {{#callable:key_bindings_get_table}} -->
The `key_bindings_get_table` function retrieves or creates a key table by name, optionally creating it if it doesn't exist.
- **Inputs**:
    - `name`: A string representing the name of the key table to retrieve or create.
    - `create`: An integer flag indicating whether to create the table if it does not exist (non-zero to create, zero otherwise).
- **Control Flow**:
    - Initialize a `key_table` structure `table_find` with the given `name`.
    - Search for an existing key table in the `key_tables` red-black tree using `RB_FIND`.
    - If the table is found or `create` is zero, return the found table or NULL.
    - If the table is not found and `create` is non-zero, allocate memory for a new `key_table`.
    - Initialize the new table's `name` and key bindings red-black trees.
    - Set the table's reference count to 1 and insert it into the `key_tables` tree.
    - Return the newly created table.
- **Output**:
    - Returns a pointer to the `key_table` structure, either found or newly created, or NULL if not found and not created.


---
### key_bindings_first_table<!-- {{#callable:key_bindings_first_table}} -->
The function `key_bindings_first_table` retrieves the first key table from a collection of key tables.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls `RB_MIN` with `key_tables` as the argument to find the minimum (first) element in the red-black tree of key tables.
    - The function returns the result of the `RB_MIN` call, which is the first key table in the collection.
- **Output**:
    - A pointer to the first `key_table` structure in the `key_tables` red-black tree, or `NULL` if the tree is empty.


---
### key_bindings_next_table<!-- {{#callable:key_bindings_next_table}} -->
The `key_bindings_next_table` function retrieves the next key table in the red-black tree of key tables.
- **Inputs**:
    - `table`: A pointer to the current key table from which the next table is to be retrieved.
- **Control Flow**:
    - The function calls `RB_NEXT` with the `key_tables` tree and the current `table` to find the next key table in the sequence.
- **Output**:
    - A pointer to the next `key_table` in the red-black tree, or `NULL` if there is no next table.


---
### key_bindings_unref_table<!-- {{#callable:key_bindings_unref_table}} -->
The `key_bindings_unref_table` function decrements the reference count of a key table and frees its resources if the count reaches zero.
- **Inputs**:
    - `table`: A pointer to a `key_table` structure whose reference count is to be decremented and potentially freed.
- **Control Flow**:
    - Decrement the reference count of the `table` by one.
    - Check if the reference count is not zero; if so, return immediately without further action.
    - Iterate over each `key_binding` in the `table->key_bindings` using `RB_FOREACH_SAFE`, remove it from the tree, and free its resources using [`key_bindings_free`](#key_bindings_free).
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
- **Output**:
    - Returns a pointer to the `key_binding` structure if found, otherwise returns `NULL`.


---
### key_bindings_get_default<!-- {{#callable:key_bindings_get_default}} -->
The `key_bindings_get_default` function retrieves a default key binding from a specified key table using a given key code.
- **Inputs**:
    - `table`: A pointer to a `key_table` structure, which contains the default key bindings.
    - `key`: A `key_code` representing the key for which the default binding is to be retrieved.
- **Control Flow**:
    - A `key_binding` structure `bd` is initialized with the provided `key`.
    - The function uses `RB_FIND` to search for the `key_binding` in the `default_key_bindings` red-black tree of the provided `table`.
    - The result of the `RB_FIND` operation, which is a pointer to the found `key_binding` or `NULL` if not found, is returned.
- **Output**:
    - A pointer to the `key_binding` structure if found, otherwise `NULL`.


---
### key_bindings_first<!-- {{#callable:key_bindings_first}} -->
The `key_bindings_first` function retrieves the first key binding from a given key table.
- **Inputs**:
    - `table`: A pointer to a `key_table` structure from which the first key binding is to be retrieved.
- **Control Flow**:
    - The function calls `RB_MIN` macro with `key_bindings` and the address of the `key_bindings` field of the `table` structure to find the minimum (first) key binding in the red-black tree.
- **Output**:
    - Returns a pointer to the first `key_binding` structure in the specified key table, or `NULL` if the table is empty.


---
### key_bindings_next<!-- {{#callable:key_bindings_next}} -->
The `key_bindings_next` function retrieves the next key binding in a red-black tree structure from a given key binding.
- **Inputs**:
    - `table`: A pointer to a `key_table` structure, which contains the red-black tree of key bindings; this parameter is unused in the function.
    - `bd`: A pointer to a `key_binding` structure, representing the current key binding from which the next binding is to be retrieved.
- **Control Flow**:
    - The function calls `RB_NEXT`, a macro or function that navigates the red-black tree to find the next node (key binding) after the given `bd` node in the `key_bindings` tree of the `table` structure.
    - The `table` parameter is marked as unused, indicating it is not directly utilized in the function's logic.
- **Output**:
    - Returns a pointer to the next `key_binding` structure in the red-black tree, or `NULL` if there is no next binding.


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
    - Retrieve or create the key table with the specified name using [`key_bindings_get_table`](#key_bindings_get_table).
    - Check if a key binding for the specified key already exists in the table using [`key_bindings_get`](#key_bindings_get).
    - If `cmdlist` is NULL and a binding exists, update the note and repeat flag of the existing binding, then return.
    - If a binding exists and `cmdlist` is not NULL, remove the existing binding from the table and free its resources.
    - Allocate memory for a new key binding and set its key, note, and command list.
    - Insert the new key binding into the table's key bindings.
    - If the repeat flag is set, update the binding's flags to include `KEY_BINDING_REPEAT`.
    - Log the new or updated key binding for debugging purposes.
- **Output**:
    - The function does not return a value; it modifies the key table by adding or updating a key binding.
- **Functions called**:
    - [`key_bindings_get_table`](#key_bindings_get_table)
    - [`key_bindings_get`](#key_bindings_get)
    - [`key_bindings_free`](#key_bindings_free)


---
### key_bindings_remove<!-- {{#callable:key_bindings_remove}} -->
The `key_bindings_remove` function removes a key binding from a specified key table and cleans up the table if it becomes empty.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the key table from which the key binding should be removed.
    - `key`: A key_code representing the key for which the binding should be removed, with any flags masked out.
- **Control Flow**:
    - Retrieve the key table associated with the given name using [`key_bindings_get_table`](#key_bindings_get_table) with the create flag set to 0.
    - If the table is NULL, return immediately as there is nothing to remove.
    - Retrieve the key binding from the table using [`key_bindings_get`](#key_bindings_get), masking out any flags from the key.
    - If the key binding is NULL, return immediately as there is nothing to remove.
    - Log the removal of the key binding for debugging purposes.
    - Remove the key binding from the table's key bindings using `RB_REMOVE`.
    - Free the memory associated with the key binding using [`key_bindings_free`](#key_bindings_free).
    - Check if both the key bindings and default key bindings of the table are empty.
    - If both are empty, remove the table from the global key tables and unreference it using [`key_bindings_unref_table`](#key_bindings_unref_table).
- **Output**:
    - The function does not return any value; it performs operations to remove a key binding and potentially a key table.
- **Functions called**:
    - [`key_bindings_get_table`](#key_bindings_get_table)
    - [`key_bindings_get`](#key_bindings_get)
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
    - If the current binding is not found, return immediately.
    - Retrieve the default key binding for the specified key using [`key_bindings_get_default`](#key_bindings_get_default).
    - If the default binding is not found, remove the current binding using [`key_bindings_remove`](#key_bindings_remove) and return.
    - Free the command list of the current binding and replace it with the command list from the default binding, incrementing the reference count.
    - Free the note of the current binding and copy the note from the default binding if it exists, otherwise set it to NULL.
    - Copy the flags from the default binding to the current binding.
- **Output**:
    - The function does not return a value; it modifies the key binding in place or removes it if no default exists.
- **Functions called**:
    - [`key_bindings_get_table`](#key_bindings_get_table)
    - [`key_bindings_get`](#key_bindings_get)
    - [`key_bindings_get_default`](#key_bindings_get_default)
    - [`key_bindings_remove`](#key_bindings_remove)


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
    - For each client using the removed table, set their key table to NULL using `server_client_set_key_table`.
- **Output**:
    - The function does not return a value; it performs operations to remove a key table and update clients accordingly.
- **Functions called**:
    - [`key_bindings_get_table`](#key_bindings_get_table)
    - [`key_bindings_unref_table`](#key_bindings_unref_table)


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
The `key_bindings_init_done` function duplicates existing key bindings into default key bindings for each key table.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which is unused in this function.
    - `data`: A void pointer to any additional data, which is unused in this function.
- **Control Flow**:
    - Iterate over each key table in the global `key_tables` red-black tree.
    - For each key table, iterate over each key binding in the `key_bindings` red-black tree.
    - Allocate memory for a new key binding and copy the key, note, flags, and command list from the existing key binding.
    - Increment the reference count of the command list associated with the key binding.
    - Insert the new key binding into the `default_key_bindings` red-black tree of the current key table.
    - Return `CMD_RETURN_NORMAL` to indicate successful completion.
- **Output**:
    - The function returns `CMD_RETURN_NORMAL`, indicating successful completion of the initialization process.


---
### key_bindings_init<!-- {{#callable:key_bindings_init}} -->
The `key_bindings_init` function initializes default key bindings for a terminal multiplexer by parsing and appending them to a command queue.
- **Inputs**:
    - None
- **Control Flow**:
    - A static array `defaults` is defined containing strings that represent default key bindings and their associated commands.
    - A loop iterates over each element in the `defaults` array.
    - For each element, the string is parsed into a command list using `cmd_parse_from_string`.
    - If parsing is successful, the command list is appended to the command queue using `cmdq_append`.
    - If parsing fails, an error is logged and the program exits with a fatal error message.
    - After processing all default key bindings, a callback `key_bindings_init_done` is appended to the command queue.
- **Output**:
    - The function does not return any value; it initializes key bindings by appending parsed commands to a command queue.


---
### key_bindings_read_only<!-- {{#callable:key_bindings_read_only}} -->
The `key_bindings_read_only` function logs an error message indicating that the client is read-only and returns an error status.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing the command queue item that triggered this function.
    - `data`: A void pointer, marked as unused, which is typically used for passing additional data but is not utilized in this function.
- **Control Flow**:
    - The function calls `cmdq_error` with the `item` and a message indicating that the client is read-only.
    - The function returns `CMD_RETURN_ERROR`, indicating an error status.
- **Output**:
    - The function returns an enum value `CMD_RETURN_ERROR`, indicating that an error occurred.


---
### key_bindings_dispatch<!-- {{#callable:key_bindings_dispatch}} -->
The `key_bindings_dispatch` function processes a key binding event and queues the corresponding command for execution, considering client read-only status and repeat flags.
- **Inputs**:
    - `bd`: A pointer to a `struct key_binding` representing the key binding to be dispatched.
    - `item`: A pointer to a `struct cmdq_item` representing the current command queue item, or NULL if there is no current item.
    - `c`: A pointer to a `struct client` representing the client for which the key binding is being dispatched.
    - `event`: A pointer to a `struct key_event` representing the key event that triggered the dispatch.
    - `fs`: A pointer to a `struct cmd_find_state` representing the command find state.
- **Control Flow**:
    - Check if the client `c` is NULL or not read-only; if so, set `readonly` to 1, otherwise check if all commands in `bd->cmdlist` are read-only and set `readonly` accordingly.
    - If `readonly` is false, create a new command queue item with a callback to `key_bindings_read_only`.
    - If `readonly` is true, check if the key binding has the `KEY_BINDING_REPEAT` flag and set the `CMDQ_STATE_REPEAT` flag if so.
    - Create a new command queue state using `cmdq_new_state` with the provided `fs`, `event`, and flags, and get a new command queue item using `cmdq_get_command`.
    - Free the newly created command queue state using `cmdq_free_state`.
    - If `item` is not NULL, insert the new item after `item` in the command queue using `cmdq_insert_after`; otherwise, append the new item to the client's command queue using `cmdq_append`.
- **Output**:
    - Returns a pointer to the newly created `struct cmdq_item` that was inserted or appended to the command queue.


# Function Declarations (Public API)

---
### key_bindings_cmp<!-- {{#callable_declaration:key_bindings_cmp}} -->
Compares two key bindings based on their key values.
- **Description**: Use this function to compare two key bindings by their key values, which is useful for sorting or ordering key bindings. The function returns a negative value if the first key binding's key is less than the second's, a positive value if it is greater, and zero if they are equal. This function is typically used in contexts where key bindings need to be organized or searched efficiently.
- **Inputs**:
    - `bd1`: A pointer to the first key_binding structure to compare. Must not be null.
    - `bd2`: A pointer to the second key_binding structure to compare. Must not be null.
- **Output**: An integer less than, equal to, or greater than zero, indicating the relative order of the key bindings based on their key values.
- **See also**: [`key_bindings_cmp`](#key_bindings_cmp)  (Implementation)


---
### key_table_cmp<!-- {{#callable_declaration:key_table_cmp}} -->
Compares two key tables by their names.
- **Description**: Use this function to compare two key tables based on their names. It is useful when you need to determine the order of key tables, for example, when sorting or searching within a collection of key tables. The function assumes that both key tables have valid, non-null name fields. It returns an integer less than, equal to, or greater than zero if the name of the first key table is found, respectively, to be less than, to match, or be greater than the name of the second key table.
- **Inputs**:
    - `table1`: A pointer to the first key_table structure to compare. Must not be null and must have a valid name field.
    - `table2`: A pointer to the second key_table structure to compare. Must not be null and must have a valid name field.
- **Output**: An integer less than, equal to, or greater than zero if the name of table1 is found, respectively, to be less than, to match, or be greater than the name of table2.
- **See also**: [`key_table_cmp`](#key_table_cmp)  (Implementation)


