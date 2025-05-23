# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it manages a collection of paste buffers. The primary functionality of this code is to handle operations related to paste buffers, which are used to store and manage text data that can be copied and pasted within tmux sessions. The code defines a `paste_buffer` structure that includes fields for storing the buffer's data, size, name, creation time, and order. It uses Red-Black trees to efficiently manage and sort these buffers by name and time, allowing for quick retrieval and manipulation.

The file provides a set of functions to perform various operations on paste buffers, such as adding new buffers, retrieving buffers by name or order, renaming buffers, and freeing them when no longer needed. It also includes functions to handle automatic buffer management, ensuring that the number of automatic buffers does not exceed a specified limit. The code is designed to be integrated into the larger tmux application, providing a specific and narrow functionality focused on paste buffer management. It does not define public APIs or external interfaces directly, but rather serves as an internal component of the tmux system, facilitating text manipulation within the application.
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
- **Description**: The `paste_num_automatic` variable is a static unsigned integer that keeps track of the number of automatic paste buffers currently in use. It is part of a system managing paste buffers, which are used to store data temporarily for pasting operations.
- **Use**: This variable is incremented when a new automatic paste buffer is added and decremented when an automatic paste buffer is freed.


---
### paste_by_name
- **Type**: `RB_HEAD(paste_name_tree, paste_buffer)`
- **Description**: The `paste_by_name` variable is a static global variable that represents a red-black tree data structure, specifically designed to store and manage `paste_buffer` structures indexed by their names. This tree allows for efficient insertion, deletion, and lookup operations based on the name of the paste buffers.
- **Use**: This variable is used to organize and access paste buffers by their names, enabling quick retrieval and management of these buffers within the application.


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
- **Description**: The `paste_buffer` structure is used to manage a collection of paste buffers in a system, such as a terminal multiplexer. Each buffer contains data that can be pasted, along with metadata such as its size, name, creation time, and order. The structure also includes entries for organizing buffers in Red-Black trees by name and time, allowing efficient retrieval and sorting. The `automatic` flag indicates whether the buffer was created automatically, which affects how it is managed and potentially replaced when limits are reached.


# Functions

---
### paste_cmp_names<!-- {{#callable:paste_cmp_names}} -->
The `paste_cmp_names` function compares the names of two paste buffers and returns the result of the comparison.
- **Inputs**:
    - `a`: A pointer to the first paste_buffer structure to be compared.
    - `b`: A pointer to the second paste_buffer structure to be compared.
- **Control Flow**:
    - The function calls the `strcmp` function with the `name` fields of the two `paste_buffer` structures `a` and `b`.
    - The result of `strcmp` is returned, which indicates the lexicographical order of the two names.
- **Output**:
    - The function returns an integer that is less than, equal to, or greater than zero if the name of `a` is found, respectively, to be less than, to match, or be greater than the name of `b`.


---
### paste_cmp_times<!-- {{#callable:paste_cmp_times}} -->
The `paste_cmp_times` function compares two paste buffers based on their order values to determine their relative order in a sorted list.
- **Inputs**:
    - `a`: A pointer to the first paste_buffer structure to be compared.
    - `b`: A pointer to the second paste_buffer structure to be compared.
- **Control Flow**:
    - The function first checks if the order of paste buffer 'a' is greater than that of 'b'.
    - If 'a' has a greater order, the function returns -1, indicating 'a' should come before 'b'.
    - If 'a' has a lesser order than 'b', the function returns 1, indicating 'b' should come before 'a'.
    - If both have the same order, the function returns 0, indicating they are equal in terms of order.
- **Output**:
    - The function returns an integer: -1 if 'a' should come before 'b', 1 if 'b' should come before 'a', and 0 if they are equal in order.


---
### paste_buffer_name<!-- {{#callable:paste_buffer_name}} -->
The `paste_buffer_name` function retrieves the name of a given paste buffer.
- **Inputs**:
    - `pb`: A pointer to a `struct paste_buffer` from which the name is to be retrieved.
- **Control Flow**:
    - The function takes a single argument, a pointer to a `struct paste_buffer`.
    - It directly returns the `name` field of the provided `paste_buffer` structure.
- **Output**:
    - The function returns a `const char *`, which is the name of the paste buffer.


---
### paste_buffer_order<!-- {{#callable:paste_buffer_order}} -->
The function `paste_buffer_order` retrieves the order value of a given paste buffer.
- **Inputs**:
    - `pb`: A pointer to a `paste_buffer` structure from which the order value is to be retrieved.
- **Control Flow**:
    - The function directly accesses the `order` field of the `paste_buffer` structure pointed to by `pb`.
    - It returns the value of the `order` field.
- **Output**:
    - The function returns an unsigned integer (`u_int`) representing the order of the paste buffer.


---
### paste_buffer_created<!-- {{#callable:paste_buffer_created}} -->
The `paste_buffer_created` function retrieves the creation time of a given paste buffer.
- **Inputs**:
    - `pb`: A pointer to a `struct paste_buffer` from which the creation time is to be retrieved.
- **Control Flow**:
    - The function directly returns the `created` field of the `paste_buffer` structure pointed to by `pb`.
- **Output**:
    - The function returns a `time_t` value representing the creation time of the specified paste buffer.


---
### paste_buffer_data<!-- {{#callable:paste_buffer_data}} -->
The `paste_buffer_data` function retrieves the data from a given paste buffer and optionally returns its size.
- **Inputs**:
    - `pb`: A pointer to a `paste_buffer` structure from which data is to be retrieved.
    - `size`: A pointer to a `size_t` variable where the size of the data will be stored, if not NULL.
- **Control Flow**:
    - Check if the `size` pointer is not NULL.
    - If `size` is not NULL, assign the size of the paste buffer's data to the dereferenced `size` pointer.
    - Return the data from the paste buffer.
- **Output**:
    - Returns a pointer to the data stored in the paste buffer.


---
### paste_walk<!-- {{#callable:paste_walk}} -->
The `paste_walk` function iterates through paste buffers in order of their creation time, starting from the earliest if no buffer is provided.
- **Inputs**:
    - `pb`: A pointer to a `paste_buffer` structure, representing the current buffer in the iteration; if NULL, the iteration starts from the earliest buffer.
- **Control Flow**:
    - Check if the input `pb` is NULL.
    - If `pb` is NULL, return the earliest paste buffer using `RB_MIN` on the `paste_time_tree`.
    - If `pb` is not NULL, return the next paste buffer in the order using `RB_NEXT` on the `paste_time_tree`.
- **Output**:
    - Returns a pointer to the next `paste_buffer` in the order of their creation time, or the earliest buffer if the input is NULL.


---
### paste_is_empty<!-- {{#callable:paste_is_empty}} -->
The `paste_is_empty` function checks if the paste buffer tree is empty.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls `RB_ROOT` with the `paste_by_time` tree to get the root node of the red-black tree.
    - It checks if the root node is `NULL`, indicating that the tree is empty.
- **Output**:
    - The function returns an integer value, `1` if the paste buffer tree is empty (i.e., the root is `NULL`), and `0` otherwise.


---
### paste_get_top<!-- {{#callable:paste_get_top}} -->
The `paste_get_top` function retrieves the most recent automatic paste buffer from a tree of paste buffers, optionally returning its name.
- **Inputs**:
    - `name`: A pointer to a constant character pointer where the name of the paste buffer will be stored if not NULL.
- **Control Flow**:
    - Initialize a pointer `pb` to the minimum element in the `paste_by_time` tree using `RB_MIN`.
    - Iterate through the paste buffers using `RB_NEXT` until an automatic buffer is found or the end of the tree is reached.
    - If an automatic buffer is found and `name` is not NULL, set `*name` to the buffer's name.
    - Return the found automatic paste buffer or NULL if none is found.
- **Output**:
    - Returns a pointer to the most recent automatic paste buffer, or NULL if no such buffer exists.


---
### paste_get_name<!-- {{#callable:paste_get_name}} -->
The `paste_get_name` function retrieves a paste buffer by its name from a red-black tree.
- **Inputs**:
    - `name`: A pointer to a constant character string representing the name of the paste buffer to be retrieved.
- **Control Flow**:
    - Check if the input `name` is NULL or an empty string; if so, return NULL.
    - Initialize a `paste_buffer` structure `pbfind` and set its `name` field to the input `name`.
    - Use the `RB_FIND` macro to search for a paste buffer with the specified name in the `paste_name_tree` red-black tree and return the result.
- **Output**:
    - Returns a pointer to the `paste_buffer` structure if found, or NULL if the buffer with the specified name does not exist or if the input name is invalid.


---
### paste_free<!-- {{#callable:paste_free}} -->
The `paste_free` function deallocates a paste buffer and removes it from the relevant data structures.
- **Inputs**:
    - `pb`: A pointer to the paste_buffer structure that is to be freed.
- **Control Flow**:
    - Call `notify_paste_buffer` with the buffer's name and a flag set to 1 to notify that the buffer is being freed.
    - Remove the buffer from the `paste_name_tree` and `paste_time_tree` red-black trees using `RB_REMOVE`.
    - Check if the buffer is marked as automatic; if so, decrement the global `paste_num_automatic` counter.
    - Free the memory allocated for the buffer's data, name, and the buffer structure itself using `free`.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided paste buffer.


---
### paste_add<!-- {{#callable:paste_add}} -->
The `paste_add` function adds a new automatic paste buffer to a global collection, ensuring the buffer limit is not exceeded by removing the oldest automatic buffers if necessary.
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
    - Generate a unique name for the new buffer by appending an index to the prefix, incrementing the index until a unique name is found.
    - Assign the provided data and size to the new paste buffer.
    - Mark the buffer as automatic and increment the count of automatic buffers.
    - Set the creation time and order for the new buffer.
    - Insert the new buffer into both the name and time red-black trees.
    - Notify that a new paste buffer has been added.
- **Output**:
    - The function does not return a value, but it modifies global data structures to include the new paste buffer.
- **Functions called**:
    - [`paste_free`](#paste_free)
    - [`paste_get_name`](#paste_get_name)


---
### paste_rename<!-- {{#callable:paste_rename}} -->
The `paste_rename` function renames a paste buffer from an old name to a new name, handling potential conflicts and updating relevant data structures.
- **Inputs**:
    - `oldname`: The current name of the paste buffer to be renamed.
    - `newname`: The new name to assign to the paste buffer.
    - `cause`: A pointer to a string that will store an error message if the function fails.
- **Control Flow**:
    - Initialize the `cause` pointer to NULL if it is not NULL.
    - Check if `oldname` is NULL or empty; if so, set `cause` to "no buffer" and return -1.
    - Check if `newname` is NULL or empty; if so, set `cause` to "new name is empty" and return -1.
    - Retrieve the paste buffer associated with `oldname` using [`paste_get_name`](#paste_get_name); if not found, set `cause` to "no buffer <oldname>" and return -1.
    - Retrieve the paste buffer associated with `newname` using [`paste_get_name`](#paste_get_name); if it is the same as the old buffer, return 0.
    - If a buffer with `newname` exists and is different, free it using [`paste_free`](#paste_free).
    - Remove the old buffer from the `paste_name_tree` red-black tree.
    - Free the old buffer's name and assign it the new name using `xstrdup`.
    - If the buffer was automatic, decrement `paste_num_automatic` and set `automatic` to 0.
    - Insert the renamed buffer back into the `paste_name_tree`.
    - Notify about the buffer renaming using `notify_paste_buffer` for both old and new names.
    - Return 0 to indicate success.
- **Output**:
    - Returns 0 on success, or -1 on failure, with an optional error message set in `cause`.
- **Functions called**:
    - [`paste_get_name`](#paste_get_name)
    - [`paste_free`](#paste_free)


---
### paste_set<!-- {{#callable:paste_set}} -->
The `paste_set` function adds or replaces a paste buffer with specified data and name, handling memory allocation and insertion into data structures.
- **Inputs**:
    - `data`: A pointer to the data to be stored in the paste buffer.
    - `size`: The size of the data to be stored.
    - `name`: The name of the paste buffer; if NULL, an automatic buffer is created.
    - `cause`: A pointer to a string that will store an error message if the function fails.
- **Control Flow**:
    - Initialize the `cause` pointer to NULL if it is provided.
    - Check if `size` is zero; if so, free the `data` and return 0.
    - If `name` is NULL, call [`paste_add`](#paste_add) to create an automatic buffer and return 0.
    - Check if `name` is an empty string; if so, set `cause` to "empty buffer name" and return -1.
    - Allocate memory for a new `paste_buffer` structure and set its fields with the provided data, size, and name.
    - Set the `automatic` flag to 0 and assign a unique order number to the buffer.
    - Record the current time as the buffer's creation time.
    - Check if a buffer with the same name already exists; if so, free the old buffer.
    - Insert the new buffer into the `paste_name_tree` and `paste_time_tree` data structures.
    - Notify that a paste buffer has been set with the given name.
    - Return 0 to indicate success.
- **Output**:
    - Returns 0 on success, or -1 if the buffer name is empty, with an optional error message set in `cause`.
- **Functions called**:
    - [`paste_add`](#paste_add)
    - [`paste_get_name`](#paste_get_name)
    - [`paste_free`](#paste_free)


---
### paste_replace<!-- {{#callable:paste_replace}} -->
The `paste_replace` function updates the data and size of a given paste buffer and notifies about the change.
- **Inputs**:
    - `pb`: A pointer to a `paste_buffer` structure that holds the paste buffer to be updated.
    - `data`: A pointer to a character array containing the new data to be set in the paste buffer.
    - `size`: The size of the new data to be set in the paste buffer.
- **Control Flow**:
    - Free the existing data in the paste buffer pointed to by `pb` using `free(pb->data)`.
    - Assign the new data pointer `data` to `pb->data`.
    - Set the new size `size` to `pb->size`.
    - Call `notify_paste_buffer` with the paste buffer's name and a flag value of `0` to notify about the update.
- **Output**:
    - The function does not return any value; it modifies the paste buffer in place.


---
### paste_make_sample<!-- {{#callable:paste_make_sample}} -->
The `paste_make_sample` function creates a visual representation of the start of a paste buffer, truncating and appending ellipses if necessary.
- **Inputs**:
    - `pb`: A pointer to a `paste_buffer` structure containing the data to be sampled.
- **Control Flow**:
    - Determine the length of the data to be processed, limiting it to a maximum of 200 characters.
    - Allocate memory for the output buffer, allowing space for potential visual encoding.
    - Convert the data from the paste buffer into a visually encoded string using specific flags for octal, C-style, tab, and newline representations.
    - If the original or encoded data exceeds the width limit, append '...' to indicate truncation.
    - Return the resulting string buffer.
- **Output**:
    - A dynamically allocated string containing a visually encoded sample of the paste buffer data, potentially truncated with '...'.


# Function Declarations (Public API)

---
### paste_cmp_names<!-- {{#callable_declaration:paste_cmp_names}} -->
Compare the names of two paste buffers.
- **Description**: Use this function to compare the names of two paste buffers lexicographically. It is typically used for sorting or searching operations where paste buffers are organized by their names. The function assumes that both paste buffers have valid, non-null name fields. It returns an integer less than, equal to, or greater than zero if the name of the first paste buffer is found, respectively, to be less than, to match, or be greater than the name of the second paste buffer.
- **Inputs**:
    - `a`: A pointer to the first paste buffer to compare. Must not be null and must have a valid name field.
    - `b`: A pointer to the second paste buffer to compare. Must not be null and must have a valid name field.
- **Output**: An integer less than, equal to, or greater than zero, indicating the lexicographical order of the names of the two paste buffers.
- **See also**: [`paste_cmp_names`](#paste_cmp_names)  (Implementation)


---
### paste_cmp_times<!-- {{#callable_declaration:paste_cmp_times}} -->
Compare two paste buffers based on their order.
- **Description**: Use this function to determine the relative order of two paste buffers. It is useful when you need to sort or prioritize paste buffers based on their order attribute. The function returns a negative value if the first buffer should come after the second, a positive value if it should come before, and zero if they are equal in order. This function assumes that both input pointers are valid and non-null.
- **Inputs**:
    - `a`: A pointer to the first paste buffer to compare. Must not be null.
    - `b`: A pointer to the second paste buffer to compare. Must not be null.
- **Output**: An integer indicating the relative order of the two buffers: -1 if the first buffer is greater, 1 if the second is greater, and 0 if they are equal.
- **See also**: [`paste_cmp_times`](#paste_cmp_times)  (Implementation)


