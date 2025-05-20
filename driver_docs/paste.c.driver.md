# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it manages a collection of paste buffers. The primary functionality of this code is to handle operations related to paste buffers, which are used to store and manage text data that can be copied and pasted within tmux sessions. The code defines a `paste_buffer` structure that includes fields for storing the buffer's data, size, name, creation time, and order. It uses Red-Black trees to efficiently manage and access these buffers by name and by time, allowing for quick retrieval and manipulation.

The file provides a variety of functions to interact with paste buffers, including adding new buffers, retrieving buffers by name or order, renaming buffers, and freeing them when no longer needed. It also includes utility functions to get buffer attributes such as name, order, creation time, and data. The code ensures that automatic buffers are managed within a specified limit, freeing the oldest ones when necessary. Additionally, it provides mechanisms to notify other parts of the system when buffers are added, renamed, or removed. This file is a crucial component of tmux's internal buffer management system, enabling efficient text handling and manipulation within the application.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `time.h`
- `tmux.h`


# Global Variables

---
### paste_next_index
- **Type**: `u_int`
- **Description**: The `paste_next_index` is a static unsigned integer variable used to keep track of the next available index for naming new paste buffers. It is incremented each time a new paste buffer is added to ensure unique naming.
- **Use**: This variable is used to generate unique names for new paste buffers by appending its value to a prefix.


---
### paste_next_order
- **Type**: `u_int`
- **Description**: The `paste_next_order` variable is a static unsigned integer that is used to track the order of paste buffers. It is incremented each time a new paste buffer is added to ensure that each buffer has a unique order value.
- **Use**: This variable is used to assign a unique order to each paste buffer, which helps in managing and sorting the buffers by their creation order.


---
### paste_num_automatic
- **Type**: `u_int`
- **Description**: The `paste_num_automatic` variable is a static unsigned integer that keeps track of the number of automatic paste buffers currently in use. Automatic paste buffers are those that are automatically managed by the system, as opposed to being explicitly named by the user.
- **Use**: This variable is incremented when a new automatic paste buffer is added and decremented when an automatic paste buffer is freed.


---
### paste_by_name
- **Type**: `RB_HEAD(paste_name_tree, paste_buffer)`
- **Description**: The `paste_by_name` variable is a static global variable that represents a red-black tree data structure. It is used to store and manage `paste_buffer` structures, which contain information about paste buffers, indexed by their names.
- **Use**: This variable is used to efficiently store and retrieve paste buffers by their names using a red-black tree for quick lookup.


---
### paste_by_time
- **Type**: `RB_HEAD(paste_time_tree, paste_buffer)`
- **Description**: The `paste_by_time` variable is a static Red-Black tree head that organizes `paste_buffer` structures by their creation time. It is used to manage and access paste buffers in a time-ordered manner, allowing operations such as insertion, deletion, and traversal based on the time of creation.
- **Use**: This variable is used to store and manage paste buffers in a time-ordered Red-Black tree, facilitating operations like retrieving the most recent buffer or iterating through buffers by their creation time.


# Data Structures

---
### paste_buffer
- **Type**: `struct`
- **Members**:
    - `data`: Pointer to the buffer's data, which is not necessarily a C string.
    - `size`: Size of the data in the buffer.
    - `name`: Name of the paste buffer.
    - `created`: Timestamp indicating when the buffer was created.
    - `automatic`: Flag indicating if the buffer was automatically created.
    - `order`: Order of the buffer, used for sorting.
    - `name_entry`: Red-Black tree entry for sorting buffers by name.
    - `time_entry`: Red-Black tree entry for sorting buffers by time.
- **Description**: The `paste_buffer` structure is used to represent a buffer of data in a paste buffer system, typically for managing clipboard-like functionality. It contains metadata about the buffer such as its name, creation time, and order, as well as flags for automatic management. The structure also includes entries for integration into Red-Black trees, allowing for efficient sorting and retrieval by name and time.


# Functions

---
### paste_cmp_names<!-- {{#callable:paste_cmp_names}} -->
The `paste_cmp_names` function compares the names of two paste buffers and returns the result of the comparison.
- **Inputs**:
    - `a`: A pointer to the first `paste_buffer` structure to be compared.
    - `b`: A pointer to the second `paste_buffer` structure to be compared.
- **Control Flow**:
    - The function calls `strcmp` with the `name` fields of the two `paste_buffer` structures `a` and `b`.
- **Output**:
    - The function returns an integer less than, equal to, or greater than zero if the name of `a` is found, respectively, to be less than, to match, or be greater than the name of `b`.


---
### paste_cmp_times<!-- {{#callable:paste_cmp_times}} -->
The `paste_cmp_times` function compares two `paste_buffer` structures based on their `order` field to determine their relative order in a sorted collection.
- **Inputs**:
    - `a`: A pointer to the first `paste_buffer` structure to be compared.
    - `b`: A pointer to the second `paste_buffer` structure to be compared.
- **Control Flow**:
    - The function first checks if the `order` field of `a` is greater than that of `b`; if so, it returns -1, indicating `a` should come before `b`.
    - If the `order` field of `a` is less than that of `b`, it returns 1, indicating `a` should come after `b`.
    - If neither condition is met, it returns 0, indicating `a` and `b` have the same order.
- **Output**:
    - The function returns an integer: -1 if `a` should come before `b`, 1 if `a` should come after `b`, and 0 if they are equal in order.


---
### paste_buffer_name<!-- {{#callable:paste_buffer_name}} -->
The `paste_buffer_name` function retrieves the name of a given paste buffer.
- **Inputs**:
    - `pb`: A pointer to a `struct paste_buffer` from which the name is to be retrieved.
- **Control Flow**:
    - The function takes a single argument, a pointer to a `struct paste_buffer`.
    - It returns the `name` field of the `struct paste_buffer`.
- **Output**:
    - The function returns a `const char *`, which is the name of the paste buffer.


---
### paste_buffer_order<!-- {{#callable:paste_buffer_order}} -->
The `paste_buffer_order` function retrieves the order value of a given paste buffer.
- **Inputs**:
    - `pb`: A pointer to a `paste_buffer` structure from which the order value is to be retrieved.
- **Control Flow**:
    - The function takes a single argument, a pointer to a `paste_buffer` structure.
    - It directly accesses the `order` field of the `paste_buffer` structure pointed to by `pb`.
    - The function returns the value of the `order` field.
- **Output**:
    - The function returns an unsigned integer representing the order of the paste buffer.


---
### paste_buffer_created<!-- {{#callable:paste_buffer_created}} -->
The `paste_buffer_created` function retrieves the creation time of a given paste buffer.
- **Inputs**:
    - `pb`: A pointer to a `paste_buffer` structure from which the creation time is to be retrieved.
- **Control Flow**:
    - The function directly accesses the `created` field of the `paste_buffer` structure pointed to by `pb`.
    - It returns the value of the `created` field.
- **Output**:
    - The function returns a `time_t` value representing the creation time of the specified paste buffer.


---
### paste_buffer_data<!-- {{#callable:paste_buffer_data}} -->
The `paste_buffer_data` function retrieves the data from a given paste buffer and optionally returns its size.
- **Inputs**:
    - `pb`: A pointer to a `paste_buffer` structure from which data is to be retrieved.
    - `size`: A pointer to a `size_t` variable where the size of the paste buffer data will be stored, if not NULL.
- **Control Flow**:
    - Check if the `size` pointer is not NULL.
    - If `size` is not NULL, assign the size of the paste buffer (`pb->size`) to the location pointed to by `size`.
    - Return the data from the paste buffer (`pb->data`).
- **Output**:
    - Returns a pointer to the data stored in the paste buffer.


---
### paste_walk<!-- {{#callable:paste_walk}} -->
The `paste_walk` function iterates over paste buffers in order of their creation time, starting from the earliest if no buffer is provided.
- **Inputs**:
    - `pb`: A pointer to a `paste_buffer` structure, representing the current buffer in the iteration; if NULL, iteration starts from the earliest buffer.
- **Control Flow**:
    - Check if the input `pb` is NULL.
    - If `pb` is NULL, return the earliest paste buffer using `RB_MIN` on the `paste_time_tree`.
    - If `pb` is not NULL, return the next paste buffer in the order using `RB_NEXT` on the `paste_time_tree`.
- **Output**:
    - Returns a pointer to the next `paste_buffer` in the order of their creation time, or the earliest buffer if the input is NULL.


---
### paste_is_empty<!-- {{#callable:paste_is_empty}} -->
The `paste_is_empty` function checks if the paste buffer tree, ordered by time, is empty.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls `RB_ROOT` with the `paste_by_time` tree to get the root node.
    - It checks if the root node is `NULL`, indicating the tree is empty.
    - The function returns the result of this comparison.
- **Output**:
    - The function returns an integer value: `1` if the paste buffer tree is empty, and `0` otherwise.


---
### paste_get_top<!-- {{#callable:paste_get_top}} -->
The `paste_get_top` function retrieves the most recent automatic paste buffer from a tree of paste buffers, optionally returning its name.
- **Inputs**:
    - `name`: A pointer to a constant character pointer where the name of the paste buffer will be stored if not NULL.
- **Control Flow**:
    - Initialize a pointer `pb` to the minimum element in the `paste_by_time` red-black tree.
    - Iterate through the tree using `RB_NEXT` until an automatic paste buffer is found or the end of the tree is reached.
    - If no automatic paste buffer is found, return NULL.
    - If a non-NULL `name` pointer is provided, set it to the name of the found paste buffer.
    - Return the found paste buffer.
- **Output**:
    - Returns a pointer to the most recent automatic paste buffer, or NULL if no such buffer exists.


---
### paste_get_name<!-- {{#callable:paste_get_name}} -->
The `paste_get_name` function retrieves a paste buffer by its name from a red-black tree.
- **Inputs**:
    - `name`: A pointer to a constant character string representing the name of the paste buffer to be retrieved.
- **Control Flow**:
    - Check if the input `name` is NULL or an empty string; if so, return NULL.
    - Create a `paste_buffer` structure `pbfind` and set its `name` field to the input `name`.
    - Use the `RB_FIND` macro to search for a paste buffer with the specified name in the `paste_name_tree` red-black tree and return the result.
- **Output**:
    - Returns a pointer to the `paste_buffer` structure if found, or NULL if the name is NULL, empty, or not found in the tree.


---
### paste_free<!-- {{#callable:paste_free}} -->
The `paste_free` function deallocates a paste buffer and removes it from the relevant data structures.
- **Inputs**:
    - `pb`: A pointer to a `paste_buffer` structure that is to be freed.
- **Control Flow**:
    - Call [`notify_paste_buffer`](notify.c.driver.md#notify_paste_buffer) with the buffer's name and a flag set to 1 to notify that the buffer is being freed.
    - Remove the buffer from the `paste_name_tree` and `paste_time_tree` red-black trees using `RB_REMOVE`.
    - Check if the buffer is marked as automatic; if so, decrement the `paste_num_automatic` counter.
    - Free the memory allocated for the buffer's data, name, and the buffer structure itself using `free`.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided paste buffer.
- **Functions called**:
    - [`notify_paste_buffer`](notify.c.driver.md#notify_paste_buffer)


---
### paste_add<!-- {{#callable:paste_add}} -->
The `paste_add` function adds a new automatic paste buffer to a global collection, ensuring it does not exceed a specified limit by removing the oldest automatic buffers if necessary.
- **Inputs**:
    - `prefix`: A string used as a prefix for naming the new paste buffer; defaults to "buffer" if NULL.
    - `data`: A pointer to the data to be stored in the new paste buffer.
    - `size`: The size of the data to be stored in the new paste buffer.
- **Control Flow**:
    - Check if the prefix is NULL and set it to "buffer" if so.
    - If the size is 0, free the data and return immediately.
    - Retrieve the buffer limit from global options.
    - Iterate over the paste buffers in reverse order by time, freeing the oldest automatic buffers until the number of automatic buffers is below the limit.
    - Allocate memory for a new paste buffer structure.
    - Generate a unique name for the new buffer by appending an index to the prefix, ensuring no name conflicts with existing buffers.
    - Assign the provided data and size to the new buffer, mark it as automatic, and increment the automatic buffer count.
    - Set the creation time and order for the new buffer.
    - Insert the new buffer into both the name and time red-black trees.
    - Notify that a new paste buffer has been added.
- **Output**:
    - The function does not return a value; it modifies global data structures to add a new paste buffer.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`paste_free`](#paste_free)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`paste_get_name`](#paste_get_name)
    - [`notify_paste_buffer`](notify.c.driver.md#notify_paste_buffer)


---
### paste_rename<!-- {{#callable:paste_rename}} -->
The `paste_rename` function renames a paste buffer from an old name to a new name, handling potential conflicts and errors.
- **Inputs**:
    - `oldname`: The current name of the paste buffer to be renamed.
    - `newname`: The new name to assign to the paste buffer.
    - `cause`: A pointer to a string that will store the error message if the function fails.
- **Control Flow**:
    - Initialize the `cause` pointer to NULL if it is not NULL.
    - Check if `oldname` is NULL or empty; if so, set `cause` to "no buffer" and return -1.
    - Check if `newname` is NULL or empty; if so, set `cause` to "new name is empty" and return -1.
    - Retrieve the paste buffer associated with `oldname` using [`paste_get_name`](#paste_get_name); if not found, set `cause` to "no buffer <oldname>" and return -1.
    - Retrieve the paste buffer associated with `newname` using [`paste_get_name`](#paste_get_name); if it is the same as the old buffer, return 0.
    - If a buffer with `newname` already exists, free it using [`paste_free`](#paste_free).
    - Remove the old buffer from the paste name tree.
    - Free the old buffer's name and assign it the new name using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - If the buffer was automatic, decrement the automatic buffer count and set its automatic flag to 0.
    - Insert the renamed buffer back into the paste name tree.
    - Notify the system of the buffer renaming using [`notify_paste_buffer`](notify.c.driver.md#notify_paste_buffer) for both the old and new names.
    - Return 0 to indicate success.
- **Output**:
    - Returns 0 on success, or -1 on failure, with an optional error message stored in `cause`.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`paste_get_name`](#paste_get_name)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`paste_free`](#paste_free)
    - [`notify_paste_buffer`](notify.c.driver.md#notify_paste_buffer)


---
### paste_set<!-- {{#callable:paste_set}} -->
The `paste_set` function adds or replaces a paste buffer with the given data and name, managing memory and updating internal data structures accordingly.
- **Inputs**:
    - `data`: A pointer to the character data to be stored in the paste buffer.
    - `size`: The size of the data to be stored in the paste buffer.
    - `name`: The name of the paste buffer; if NULL, an automatic buffer is created.
    - `cause`: A pointer to a string that will be set to an error message if an error occurs, or NULL if no error occurs.
- **Control Flow**:
    - Initialize the `cause` pointer to NULL if it is not NULL.
    - Check if `size` is zero; if so, free `data` and return 0.
    - Check if `name` is NULL; if so, call [`paste_add`](#paste_add) to add an automatic buffer and return 0.
    - Check if `name` is an empty string; if so, set `cause` to "empty buffer name" and return -1.
    - Allocate memory for a new `paste_buffer` structure and set its fields with the provided data, size, and name.
    - Set the `automatic` flag to 0 and assign a unique order number to the buffer.
    - Set the creation time of the buffer to the current time.
    - Check if a buffer with the same name already exists; if so, free the old buffer.
    - Insert the new buffer into the `paste_name_tree` and `paste_time_tree` red-black trees.
    - Notify that a paste buffer has been set with the given name.
    - Return 0 to indicate success.
- **Output**:
    - Returns 0 on success, or -1 if an error occurs (e.g., if the buffer name is empty).
- **Functions called**:
    - [`paste_add`](#paste_add)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`paste_get_name`](#paste_get_name)
    - [`paste_free`](#paste_free)
    - [`notify_paste_buffer`](notify.c.driver.md#notify_paste_buffer)


---
### paste_replace<!-- {{#callable:paste_replace}} -->
The `paste_replace` function updates the data and size of a given paste buffer and notifies about the change.
- **Inputs**:
    - `pb`: A pointer to a `paste_buffer` structure that represents the paste buffer to be updated.
    - `data`: A pointer to a character array containing the new data to be set in the paste buffer.
    - `size`: The size of the new data to be set in the paste buffer.
- **Control Flow**:
    - The function begins by freeing the existing data in the paste buffer pointed to by `pb` using `free(pb->data)` to prevent memory leaks.
    - It then assigns the new data pointer `data` to `pb->data`, effectively replacing the old data with the new data.
    - The size of the new data is assigned to `pb->size`, updating the size of the paste buffer.
    - Finally, the function calls `notify_paste_buffer(pb->name, 0)` to notify that the paste buffer has been updated.
- **Output**:
    - The function does not return any value; it performs its operations directly on the provided `paste_buffer` structure.
- **Functions called**:
    - [`notify_paste_buffer`](notify.c.driver.md#notify_paste_buffer)


---
### paste_make_sample<!-- {{#callable:paste_make_sample}} -->
The `paste_make_sample` function creates a visual representation of the start of a paste buffer, truncating and appending ellipses if necessary.
- **Inputs**:
    - `pb`: A pointer to a `paste_buffer` structure containing the data to be converted into a sample string.
- **Control Flow**:
    - Determine the length of the data to be processed, limiting it to a maximum of 200 characters.
    - Allocate memory for the output buffer, allowing space for potential escape sequences.
    - Convert the data from the paste buffer into a visually safe string using [`utf8_strvis`](utf8.c.driver.md#utf8_strvis), applying specific flags for character representation.
    - Check if the original or converted data exceeds the width limit, and if so, append '...' to indicate truncation.
    - Return the resulting string buffer.
- **Output**:
    - A dynamically allocated string containing a visual representation of the paste buffer's data, potentially truncated with '...' if it exceeds the specified width.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`utf8_strvis`](utf8.c.driver.md#utf8_strvis)
    - [`strlcpy`](compat/strlcpy.c.driver.md#strlcpy)


