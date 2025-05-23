# Purpose
This C source code file is part of the tmux project, specifically handling the management of options within the application. The code provides a comprehensive implementation for managing options, which are stored in a red-black tree structure. Each option has a name, type, and value, and can be organized hierarchically with parent-child relationships. The file defines various functions to create, retrieve, modify, and delete options, as well as to convert option values to and from strings. It supports different types of options, including strings, numbers, commands, and arrays, and includes mechanisms for handling default values and validating option values against predefined constraints.

The code is structured around several key data structures, such as `options`, `options_entry`, and [`options_array_item`](#options_array_item), which facilitate the storage and manipulation of options. It also includes a set of macros to determine the type of an option and functions to compare and manage the red-black trees. The file is not an executable but rather a library intended to be used within the tmux application, providing a public API for option management. It does not define external interfaces but is crucial for the internal configuration management of tmux, allowing for dynamic and flexible option handling.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `fnmatch.h`
- `stdarg.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### options_add
- **Type**: `static struct options_entry*`
- **Description**: The `options_add` function is a static function that returns a pointer to an `options_entry` structure. It is used to add a new option entry to an options tree, which is part of a red-black tree data structure used for managing options in the program.
- **Use**: This function is used to add a new option entry to the options tree, ensuring that each option has a unique name and is properly inserted into the red-black tree.


# Data Structures

---
### options_array_item
- **Type**: `struct`
- **Members**:
    - `index`: An unsigned integer representing the index of the item in the array.
    - `value`: A union representing the value of the option, which can be of various types.
    - `entry`: A red-black tree entry used to organize the items in a red-black tree.
- **Description**: The `options_array_item` structure is used to represent an individual item within an options array in the tmux codebase. Each item has an index to identify its position, a value that can be of different types as defined by the `options_value` union, and an `entry` field that allows the item to be part of a red-black tree, facilitating efficient storage and retrieval operations.


---
### options_entry
- **Type**: `struct`
- **Members**:
    - `owner`: A pointer to the options structure that owns this entry.
    - `name`: A constant character pointer representing the name of the option.
    - `tableentry`: A pointer to the options_table_entry structure that defines the option's properties.
    - `value`: A union options_value that holds the value of the option.
    - `cached`: An integer indicating whether the style is cached.
    - `style`: A style structure representing the style associated with the option.
    - `entry`: A red-black tree entry for managing the options_entry in a tree.
- **Description**: The `options_entry` structure is a component of an options management system, typically used in software like tmux, to handle configuration options. Each entry represents a single option, storing its name, value, and associated metadata. The structure is designed to be part of a red-black tree, allowing efficient management and retrieval of options. It includes a pointer to its owner options structure, a reference to its definition in an options table, and a union to store its value, which can be of various types. Additionally, it supports caching of style information for performance optimization.


---
### options
- **Type**: `struct`
- **Members**:
    - `tree`: A red-black tree head for storing options entries.
    - `parent`: A pointer to the parent options structure.
- **Description**: The `options` structure is designed to manage a collection of configuration options, each identified by a name, type, and value, organized in a red-black tree for efficient retrieval. It supports hierarchical option management by allowing each `options` instance to have a parent, enabling inheritance of options from a parent structure. This is useful in scenarios where options need to be overridden or extended in a nested configuration context.


# Functions

---
### options_array_cmp<!-- {{#callable:options_array_cmp}} -->
Compares two `options_array_item` structures based on their index values.
- **Inputs**:
    - `a1`: A pointer to the first `options_array_item` structure to compare.
    - `a2`: A pointer to the second `options_array_item` structure to compare.
- **Control Flow**:
    - The function first checks if the index of `a1` is less than that of `a2`, returning -1 if true.
    - If the index of `a1` is greater than that of `a2`, it returns 1.
    - If both indices are equal, it returns 0.
- **Output**:
    - Returns an integer indicating the comparison result: -1 if `a1` is less than `a2`, 1 if greater, and 0 if they are equal.


---
### options_cmp<!-- {{#callable:options_cmp}} -->
The `options_cmp` function compares two `options_entry` structures based on their `name` field.
- **Inputs**:
    - `lhs`: A pointer to the first `options_entry` structure to compare.
    - `rhs`: A pointer to the second `options_entry` structure to compare.
- **Control Flow**:
    - The function uses the `strcmp` function to compare the `name` fields of the two `options_entry` structures.
    - It returns the result of the comparison, which indicates the lexicographical order of the names.
- **Output**:
    - The function returns an integer: a negative value if `lhs->name` is less than `rhs->name`, zero if they are equal, and a positive value if `lhs->name` is greater than `rhs->name`.


---
### options_map_name<!-- {{#callable:options_map_name}} -->
The `options_map_name` function maps a given name to its corresponding option name if it exists in a predefined mapping.
- **Inputs**:
    - `name`: A constant character pointer representing the name to be mapped.
- **Control Flow**:
    - The function initializes a pointer `map` to iterate over the `options_other_names` mapping.
    - It enters a loop that continues until it finds a mapping where `map->from` is NULL.
    - Within the loop, it compares the `from` field of the current mapping with the input `name` using `strcmp`.
    - If a match is found, it returns the corresponding `to` field from the mapping.
    - If no match is found after the loop completes, it returns the original `name`.
- **Output**:
    - Returns a constant character pointer to the mapped name if found; otherwise, it returns the original input name.


---
### options_parent_table_entry<!-- {{#callable:options_parent_table_entry}} -->
The `options_parent_table_entry` function retrieves the table entry of a specified option from the parent options structure.
- **Inputs**:
    - `oo`: A pointer to the `options` structure from which the parent options are accessed.
    - `s`: A string representing the name of the option to retrieve from the parent options.
- **Control Flow**:
    - The function first checks if the `parent` field of the `options` structure is NULL, indicating that there are no parent options; if so, it calls `fatalx` to terminate the program with an error message.
    - Next, it calls the [`options_get`](#options_get) function to attempt to retrieve the `options_entry` associated with the specified option name `s` from the parent options.
    - If the `options_entry` is NULL, indicating that the option does not exist in the parent options, it again calls `fatalx` to terminate the program with an error message.
    - Finally, the function returns the `tableentry` field of the found `options_entry`, which contains the details of the option.
- **Output**:
    - The function returns a pointer to the `options_table_entry` associated with the specified option name from the parent options, or terminates the program if the option or parent options are not found.
- **Functions called**:
    - [`options_get`](#options_get)


---
### options_value_free<!-- {{#callable:options_value_free}} -->
Frees the memory associated with an `options_entry`'s value based on its type.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure that contains information about the option, including its type.
    - `ov`: A pointer to a union `options_value` that holds the actual value of the option, which can be of different types.
- **Control Flow**:
    - Checks if the type of the option represented by `o` is a string using the `OPTIONS_IS_STRING` macro.
    - If it is a string, it frees the memory allocated for the string in `ov->string`.
    - Checks if the type of the option is a command using the `OPTIONS_IS_COMMAND` macro.
    - If it is a command and `ov->cmdlist` is not NULL, it calls [`cmd_list_free`](cmd.c.driver.md#cmd_list_free) to free the command list.
- **Output**:
    - This function does not return a value; it performs memory deallocation for the option's value.
- **Functions called**:
    - [`cmd_list_free`](cmd.c.driver.md#cmd_list_free)


---
### options_value_to_string<!-- {{#callable:options_value_to_string}} -->
Converts an `options_entry` value to its string representation based on its type.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure that contains the option's metadata including its type.
    - `ov`: A pointer to a union `options_value` that holds the actual value of the option.
    - `numeric`: An integer flag indicating whether to return numeric representations for certain types.
- **Control Flow**:
    - Checks if the option is a command using `OPTIONS_IS_COMMAND` macro and returns its string representation using [`cmd_list_print`](cmd.c.driver.md#cmd_list_print).
    - If the option is a number, it checks the specific type of the number (e.g., number, key, color, flag, choice) and formats the output accordingly.
    - For string options, it directly duplicates the string value.
    - If none of the conditions are met, it returns an empty string.
- **Output**:
    - Returns a dynamically allocated string representation of the option's value, or an empty string if the option type is unrecognized.
- **Functions called**:
    - [`cmd_list_print`](cmd.c.driver.md#cmd_list_print)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)
    - [`colour_tostring`](colour.c.driver.md#colour_tostring)


---
### options_create<!-- {{#callable:options_create}} -->
Creates a new `options` structure with an optional parent.
- **Inputs**:
    - `parent`: A pointer to an existing `options` structure that serves as the parent of the new options.
- **Control Flow**:
    - Allocates memory for a new `options` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc).
    - Initializes the red-black tree within the new `options` structure using `RB_INIT`.
    - Sets the `parent` field of the new `options` structure to the provided `parent` argument.
    - Returns a pointer to the newly created `options` structure.
- **Output**:
    - Returns a pointer to the newly created `options` structure.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### options_free<!-- {{#callable:options_free}} -->
The `options_free` function deallocates memory associated with an `options` structure and its entries.
- **Inputs**:
    - `oo`: A pointer to an `options` structure that contains a red-black tree of options entries to be freed.
- **Control Flow**:
    - The function uses `RB_FOREACH_SAFE` to iterate over each `options_entry` in the red-black tree associated with the `options` structure.
    - For each entry, it calls the [`options_remove`](#options_remove) function to remove and free the entry.
    - After all entries have been processed, it frees the memory allocated for the `options` structure itself.
- **Output**:
    - The function does not return a value; it performs memory deallocation and cleanup.
- **Functions called**:
    - [`options_remove`](#options_remove)


---
### options_get_parent<!-- {{#callable:options_get_parent}} -->
The `options_get_parent` function retrieves the parent `options` structure of a given `options` instance.
- **Inputs**:
    - `oo`: A pointer to an `options` structure whose parent is to be retrieved.
- **Control Flow**:
    - The function directly accesses the `parent` member of the `options` structure pointed to by `oo`.
    - It returns the value of the `parent` member, which is also of type `struct options *`.
- **Output**:
    - Returns a pointer to the parent `options` structure, or NULL if the parent does not exist.


---
### options_set_parent<!-- {{#callable:options_set_parent}} -->
Sets the parent options structure for a given options instance.
- **Inputs**:
    - `oo`: A pointer to the `options` structure that is being modified.
    - `parent`: A pointer to the `options` structure that will be set as the parent.
- **Control Flow**:
    - The function directly assigns the `parent` pointer of the `options` structure `oo` to the provided `parent` pointer.
- **Output**:
    - This function does not return a value; it modifies the `parent` field of the `options` structure.


---
### options_first<!-- {{#callable:options_first}} -->
The `options_first` function retrieves the first entry in a red-black tree of options.
- **Inputs**:
    - `oo`: A pointer to a `struct options` which contains a red-black tree of options entries.
- **Control Flow**:
    - The function calls `RB_MIN` to find the minimum (first) entry in the red-black tree associated with the `options` structure.
    - It returns the result of the `RB_MIN` function, which is either a pointer to the first `options_entry` or NULL if the tree is empty.
- **Output**:
    - The function returns a pointer to the first `options_entry` in the tree, or NULL if the tree is empty.


---
### options_next<!-- {{#callable:options_next}} -->
The `options_next` function retrieves the next `options_entry` in a red-black tree.
- **Inputs**:
    - `o`: A pointer to the current `options_entry` whose successor is to be found.
- **Control Flow**:
    - The function calls `RB_NEXT` to find the next entry in the red-black tree.
    - It uses the `options_tree` and the current entry `o` to determine the successor.
- **Output**:
    - Returns a pointer to the next `options_entry` in the tree, or NULL if there is no next entry.


---
### options_get_only<!-- {{#callable:options_get_only}} -->
Retrieves an `options_entry` from a red-black tree based on the provided name.
- **Inputs**:
    - `oo`: A pointer to an `options` structure that contains the red-black tree of options.
    - `name`: A string representing the name of the option to retrieve.
- **Control Flow**:
    - Creates a temporary `options_entry` structure with the provided name.
    - Attempts to find the entry in the red-black tree using `RB_FIND`.
    - If the entry is not found, it maps the name using [`options_map_name`](#options_map_name) and tries to find it again.
    - Returns the found entry or NULL if it does not exist.
- **Output**:
    - Returns a pointer to the `options_entry` if found, otherwise returns NULL.
- **Functions called**:
    - [`options_map_name`](#options_map_name)


---
### options_get<!-- {{#callable:options_get}} -->
The `options_get` function retrieves an `options_entry` by name from a given `options` structure, searching through parent options if necessary.
- **Inputs**:
    - `oo`: A pointer to the `options` structure from which to retrieve the option.
    - `name`: A string representing the name of the option to retrieve.
- **Control Flow**:
    - The function first attempts to find the option using the [`options_get_only`](#options_get_only) function.
    - If the option is not found (i.e., `o` is NULL`), it traverses up the parent options structure.
    - For each parent, it again attempts to find the option until either the option is found or there are no more parents to check.
- **Output**:
    - Returns a pointer to the `options_entry` corresponding to the specified name, or NULL if the option is not found in the current or any parent options.
- **Functions called**:
    - [`options_get_only`](#options_get_only)


---
### options_empty<!-- {{#callable:options_empty}} -->
The `options_empty` function initializes a new `options_entry` with a specified name and associates it with a given options table entry.
- **Inputs**:
    - `oo`: A pointer to an `options` structure representing the current options context.
    - `oe`: A pointer to an `options_table_entry` structure that contains metadata about the option being added.
- **Control Flow**:
    - Calls the [`options_add`](#options_add) function to create a new `options_entry` with the name specified in `oe`.
    - Assigns the `tableentry` field of the newly created `options_entry` to point to the provided `options_table_entry`.
    - Checks if the `flags` field of `oe` indicates that the option is an array; if so, initializes the red-black tree for the array values.
- **Output**:
    - Returns a pointer to the newly created `options_entry`.
- **Functions called**:
    - [`options_add`](#options_add)


---
### options_default<!-- {{#callable:options_default}} -->
The `options_default` function initializes an `options_entry` structure with default values based on the provided options table entry.
- **Inputs**:
    - `oo`: A pointer to an `options` structure that represents the current options context.
    - `oe`: A pointer to an `options_table_entry` structure that contains the default values and metadata for the option being initialized.
- **Control Flow**:
    - Calls [`options_empty`](#options_empty) to create an empty `options_entry` and associate it with the provided options context.
    - Checks if the option is an array by evaluating the `OPTIONS_TABLE_IS_ARRAY` flag in `oe->flags`.
    - If it is an array and `oe->default_arr` is NULL, it assigns a default string value using [`options_array_assign`](#options_array_assign).
    - If `oe->default_arr` is not NULL, it iterates through the array and sets each default value using [`options_array_set`](#options_array_set).
    - If the option is not an array, it checks the type of the option (`OPTIONS_TABLE_STRING`, `OPTIONS_TABLE_COMMAND`, or others) and assigns the appropriate default value to the `options_entry`'s value union.
- **Output**:
    - Returns a pointer to the initialized `options_entry` structure containing the default values.
- **Functions called**:
    - [`options_empty`](#options_empty)
    - [`options_array_assign`](#options_array_assign)
    - [`options_array_set`](#options_array_set)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### options_default_to_string<!-- {{#callable:options_default_to_string}} -->
Converts the default value of an options table entry to a string representation.
- **Inputs**:
    - `oe`: A pointer to a `struct options_table_entry` that contains the type and default value of the option.
- **Control Flow**:
    - The function begins by checking the `type` of the option represented by `oe`.
    - Depending on the `type`, it handles the conversion of the default value to a string in various ways, such as duplicating strings, formatting numbers, or looking up keys and colors.
    - If the `type` does not match any known types, it triggers a fatal error indicating an unknown option type.
    - Finally, it returns the resulting string.
- **Output**:
    - Returns a pointer to a dynamically allocated string that represents the default value of the option, or triggers a fatal error if the option type is unknown.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)
    - [`colour_tostring`](colour.c.driver.md#colour_tostring)


---
### options_add<!-- {{#callable:options_add}} -->
The `options_add` function adds a new option entry to a given options structure.
- **Inputs**:
    - `oo`: A pointer to the `options` structure where the new option will be added.
    - `name`: A string representing the name of the option to be added.
- **Control Flow**:
    - The function first checks if an option with the same name already exists in the options structure by calling [`options_get_only`](#options_get_only).
    - If an existing option is found, it is removed using [`options_remove`](#options_remove).
    - A new `options_entry` structure is allocated and initialized with the owner and name.
    - The new option entry is then inserted into the red-black tree of options using `RB_INSERT`.
    - Finally, the function returns a pointer to the newly created `options_entry`.
- **Output**:
    - The function returns a pointer to the newly created `options_entry` structure representing the added option.
- **Functions called**:
    - [`options_get_only`](#options_get_only)
    - [`options_remove`](#options_remove)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### options_remove<!-- {{#callable:options_remove}} -->
The `options_remove` function deallocates and removes an `options_entry` from its owner options tree.
- **Inputs**:
    - `o`: A pointer to the `options_entry` structure that is to be removed.
- **Control Flow**:
    - The function retrieves the owner of the `options_entry` from the `owner` field.
    - It checks if the `options_entry` is an array using the `OPTIONS_IS_ARRAY` macro.
    - If it is an array, it calls [`options_array_clear`](#options_array_clear) to free the array elements.
    - If it is not an array, it calls [`options_value_free`](#options_value_free) to free the value associated with the entry.
    - The entry is then removed from the red-black tree using `RB_REMOVE`.
    - Finally, it frees the memory allocated for the entry's name and the entry itself.
- **Output**:
    - The function does not return a value; it performs cleanup and deallocation of resources associated with the `options_entry`.
- **Functions called**:
    - [`options_array_clear`](#options_array_clear)
    - [`options_value_free`](#options_value_free)


---
### options_name<!-- {{#callable:options_name}} -->
The `options_name` function retrieves the name of a given options entry.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure, which contains the details of the options entry including its name.
- **Control Flow**:
    - The function directly accesses the `name` field of the `options_entry` structure pointed to by `o`.
    - It returns the value of the `name` field without any additional processing or checks.
- **Output**:
    - Returns a constant character pointer to the name of the options entry.


---
### options_owner<!-- {{#callable:options_owner}} -->
The `options_owner` function retrieves the owner of a given `options_entry`.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure whose owner is to be retrieved.
- **Control Flow**:
    - The function directly accesses the `owner` field of the `options_entry` structure.
    - It returns the value of the `owner` field, which is a pointer to an `options` structure.
- **Output**:
    - Returns a pointer to the `options` structure that owns the specified `options_entry`.


---
### options_table_entry<!-- {{#callable:options_table_entry}} -->
The `options_table_entry` function retrieves the `tableentry` field from a given `options_entry` structure.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure from which the `tableentry` is to be retrieved.
- **Control Flow**:
    - The function directly accesses the `tableentry` member of the `options_entry` structure pointed to by `o`.
    - It returns the value of `tableentry` without any additional processing or checks.
- **Output**:
    - Returns a pointer to a constant `options_table_entry` structure, which represents the table entry associated with the provided options entry.


---
### options_array_item<!-- {{#callable:options_array_item}} -->
The `options_array_item` function retrieves an `options_array_item` from a red-black tree based on the specified index.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure that contains the array from which the item is to be retrieved.
    - `idx`: An unsigned integer representing the index of the desired `options_array_item`.
- **Control Flow**:
    - A local variable `a` of type `options_array_item` is created and its `index` field is set to the provided `idx` value.
    - The function then calls `RB_FIND` to search for the `options_array_item` in the red-black tree associated with `o->value.array` using the local variable `a` as the search key.
- **Output**:
    - Returns a pointer to the found `options_array_item` if it exists, or NULL if no item with the specified index is found.


---
### options_array_new<!-- {{#callable:options_array_new}} -->
Creates a new `options_array_item` and inserts it into the options entry's array.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure that represents the options entry to which the new array item will be added.
    - `idx`: An unsigned integer representing the index at which the new array item will be inserted.
- **Control Flow**:
    - Allocates memory for a new `options_array_item` using [`xcalloc`](xmalloc.c.driver.md#xcalloc), initializing it to zero.
    - Sets the `index` field of the newly created item to the provided `idx` value.
    - Inserts the new item into the red-black tree associated with the `options_entry`'s array using `RB_INSERT`.
    - Returns a pointer to the newly created `options_array_item`.
- **Output**:
    - Returns a pointer to the newly created `options_array_item` that has been inserted into the options entry's array.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### options_array_free<!-- {{#callable:options_array_free}} -->
The `options_array_free` function deallocates memory associated with an `options_array_item` and removes it from the options entry's array.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure representing the options entry that owns the array item.
    - `a`: A pointer to an `options_array_item` structure that is to be freed.
- **Control Flow**:
    - Calls [`options_value_free`](#options_value_free) to free the value associated with the array item.
    - Removes the array item from the red-black tree that maintains the array items in the options entry.
    - Frees the memory allocated for the array item itself.
- **Output**:
    - This function does not return a value; it performs memory deallocation and structural updates.
- **Functions called**:
    - [`options_value_free`](#options_value_free)


---
### options_array_clear<!-- {{#callable:options_array_clear}} -->
The `options_array_clear` function frees all items in an options array if the provided options entry is indeed an array.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure that represents the options entry to be cleared.
- **Control Flow**:
    - The function first checks if the provided `options_entry` (`o`) is an array using the `OPTIONS_IS_ARRAY` macro.
    - If `o` is not an array, the function returns immediately without performing any operations.
    - If `o` is an array, it iterates over each item in the array using `RB_FOREACH_SAFE`, which allows safe removal of items during iteration.
    - For each item in the array, it calls the [`options_array_free`](#options_array_free) function to free the memory associated with that item.
- **Output**:
    - The function does not return a value; it modifies the state of the options entry by freeing its associated array items.
- **Functions called**:
    - [`options_array_free`](#options_array_free)


---
### options_array_get<!-- {{#callable:options_array_get}} -->
The `options_array_get` function retrieves the value of an item at a specified index from an options array.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure representing the options array from which the value is to be retrieved.
    - `idx`: An unsigned integer representing the index of the item in the options array.
- **Control Flow**:
    - The function first checks if the provided `options_entry` (`o`) is an array using the `OPTIONS_IS_ARRAY` macro.
    - If `o` is not an array, the function returns NULL.
    - Next, it attempts to retrieve the array item at the specified index using the [`options_array_item`](#options_array_item) function.
    - If the item at the specified index is NULL, the function returns NULL.
    - Finally, if the item is found, the function returns a pointer to the value of that item.
- **Output**:
    - The function returns a pointer to the `options_value` of the array item at the specified index, or NULL if the index is out of bounds or if the options entry is not an array.
- **Functions called**:
    - [`options_array_item`](#options_array_item)


---
### options_array_set<!-- {{#callable:options_array_set}} -->
Sets a value in an options array, handling different types of values and conditions.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure representing the options array.
    - `idx`: An unsigned integer index indicating the position in the options array to set.
    - `value`: A string representing the value to set at the specified index.
    - `append`: An integer flag indicating whether to append the value to the existing string at the index.
    - `cause`: A pointer to a string that will hold an error message if the operation fails.
- **Control Flow**:
    - Checks if the `options_entry` is an array; if not, sets an error message and returns -1.
    - If `value` is NULL, it frees the array item at the specified index if it exists and returns 0.
    - If the entry is a command type, it parses the command from the string and handles errors accordingly.
    - If the entry is a string type, it either appends the new value to the existing string or sets it directly.
    - If the entry is of type colour, it converts the string to a colour number and sets it.
    - If none of the conditions match, it sets an error message indicating the wrong array type and returns -1.
- **Output**:
    - Returns 0 on success, -1 on failure with an error message set in `cause` if applicable.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`options_array_item`](#options_array_item)
    - [`options_array_free`](#options_array_free)
    - [`options_array_new`](#options_array_new)
    - [`options_value_free`](#options_value_free)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`colour_fromstring`](colour.c.driver.md#colour_fromstring)


---
### options_array_assign<!-- {{#callable:options_array_assign}} -->
The `options_array_assign` function assigns values to an options array based on a given string input, splitting it by a specified separator.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure representing the options array to be modified.
    - `s`: A string containing the values to be assigned to the options array.
    - `cause`: A pointer to a character pointer that can be used to store an error message if the assignment fails.
- **Control Flow**:
    - The function first retrieves the separator from the `options_entry` structure; if none is provided, it defaults to a space or comma.
    - If the separator is an empty string and the input string is also empty, the function returns 0, indicating no action is needed.
    - If the input string is not empty, it duplicates the string for processing and begins to split it using the specified separator.
    - For each non-empty substring obtained from the split, it checks for the next available index in the options array and assigns the value using [`options_array_set`](#options_array_set).
    - If the array is full (i.e., the index reaches UINT_MAX), the loop breaks, and if any assignment fails, it frees the duplicated string and returns -1.
    - Finally, if all assignments are successful, it frees the duplicated string and returns 0.
- **Output**:
    - The function returns 0 on success, -1 if an error occurs during assignment, and does not modify the options array if the input string is empty.
- **Functions called**:
    - [`options_array_item`](#options_array_item)
    - [`options_array_set`](#options_array_set)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`strsep`](compat/strsep.c.driver.md#strsep)


---
### options_array_first<!-- {{#callable:options_array_first}} -->
The `options_array_first` function retrieves the first item from an options array.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure that represents the options array from which the first item is to be retrieved.
- **Control Flow**:
    - The function first checks if the provided `options_entry` (`o`) is an array using the `OPTIONS_IS_ARRAY` macro.
    - If `o` is not an array, the function returns `NULL`.
    - If `o` is an array, it calls `RB_MIN` to retrieve the minimum (first) item from the red-black tree representing the array.
- **Output**:
    - The function returns a pointer to the first `options_array_item` in the array if it exists, or `NULL` if the input is not an array.


---
### options_array_next<!-- {{#callable:options_array_next}} -->
The `options_array_next` function retrieves the next item in a red-black tree of options array items.
- **Inputs**:
    - `a`: A pointer to the current `options_array_item` whose successor is to be found.
- **Control Flow**:
    - The function calls `RB_NEXT` macro to find the next item in the red-black tree.
    - It uses the `options_array` tree and the current item `a` to determine the successor.
- **Output**:
    - Returns a pointer to the next `options_array_item` in the tree, or NULL if there is no next item.


---
### options_array_item_index<!-- {{#callable:options_array_item_index}} -->
The `options_array_item_index` function retrieves the index of a given `options_array_item`.
- **Inputs**:
    - `a`: A pointer to an `options_array_item` structure whose index is to be retrieved.
- **Control Flow**:
    - The function directly accesses the `index` field of the `options_array_item` structure pointed to by `a`.
    - It returns the value of the `index` field without any additional logic or conditions.
- **Output**:
    - The function returns an unsigned integer representing the index of the specified `options_array_item`.


---
### options_array_item_value<!-- {{#callable:options_array_item_value}} -->
The `options_array_item_value` function returns a pointer to the `value` field of a given `options_array_item`.
- **Inputs**:
    - `a`: A pointer to an `options_array_item` structure whose `value` field is to be accessed.
- **Control Flow**:
    - The function directly accesses the `value` field of the `options_array_item` structure pointed to by `a`.
    - It returns the address of the `value` field, which is of type `union options_value`.
- **Output**:
    - The function outputs a pointer to the `value` field of the specified `options_array_item`, allowing access to the stored options value.


---
### options_is_array<!-- {{#callable:options_is_array}} -->
The `options_is_array` function checks if a given options entry is of type array.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure that represents the options entry to be checked.
- **Control Flow**:
    - The function evaluates the macro `OPTIONS_IS_ARRAY(o)` which checks if the `tableentry` of the options entry is not NULL and if the flags indicate that it is an array.
    - The result of the evaluation is returned as an integer value.
- **Output**:
    - Returns a non-zero integer if the options entry is an array, otherwise returns zero.


---
### options_is_string<!-- {{#callable:options_is_string}} -->
The `options_is_string` function checks if a given options entry is of type string.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure that represents the options entry to be checked.
- **Control Flow**:
    - The function directly returns the result of the macro `OPTIONS_IS_STRING(o)`.
    - The macro checks if the `tableentry` of the options entry is NULL or if its type is `OPTIONS_TABLE_STRING`.
- **Output**:
    - Returns a non-zero integer if the options entry is a string type, otherwise returns zero.


---
### options_to_string<!-- {{#callable:options_to_string}} -->
Converts an `options_entry` structure to a string representation.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure that contains the options to be converted.
    - `idx`: An integer index used to specify which item in an array to convert; -1 indicates all items.
    - `numeric`: An integer flag that determines whether numeric values should be represented numerically or as strings.
- **Control Flow**:
    - Checks if the `options_entry` is an array using the `OPTIONS_IS_ARRAY` macro.
    - If `idx` is -1, iterates over all items in the array and converts each to a string using [`options_value_to_string`](#options_value_to_string), concatenating them with spaces.
    - If `idx` is valid, retrieves the specific item at that index and converts it to a string.
    - If the `options_entry` is not an array, directly converts its value to a string using [`options_value_to_string`](#options_value_to_string).
- **Output**:
    - Returns a dynamically allocated string representing the options; if no options are present, returns an empty string.
- **Functions called**:
    - [`options_value_to_string`](#options_value_to_string)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`options_array_item`](#options_array_item)


---
### options_parse<!-- {{#callable:options_parse}} -->
Parses an option name and extracts an index if present.
- **Inputs**:
    - `name`: A string representing the option name, which may include an index in square brackets.
    - `idx`: A pointer to an integer where the extracted index will be stored.
- **Control Flow**:
    - Checks if the input string `name` is empty; if so, returns NULL.
    - Duplicates the `name` string into `copy`.
    - Searches for the first occurrence of '[' in `copy` to determine if an index is specified.
    - If '[' is not found, sets `*idx` to -1 and returns the duplicated string.
    - Searches for the corresponding ']' character after the '['.
    - Validates that the character following ']' is the end of the string and that the character before ']' is a digit.
    - If the conditions are not met, frees `copy` and returns NULL.
    - Uses `sscanf` to extract the integer index from the string and stores it in `*idx`.
    - Modifies `copy` to terminate the string at the '[' character and returns it.
- **Output**:
    - Returns a pointer to the duplicated string without the index part if present, or NULL if parsing fails.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### options_parse_get<!-- {{#callable:options_parse_get}} -->
The `options_parse_get` function retrieves an `options_entry` from a given options structure based on a parsed string.
- **Inputs**:
    - `oo`: A pointer to the `options` structure from which the option is to be retrieved.
    - `s`: A string containing the name of the option to retrieve, which may include an index.
    - `idx`: A pointer to an integer that will store the index parsed from the string, if applicable.
    - `only`: An integer flag indicating whether to retrieve only the specified option without searching parent options.
- **Control Flow**:
    - The function calls [`options_parse`](#options_parse) to extract the option name from the input string `s` and updates `idx` with the parsed index.
    - If the parsed name is NULL, the function returns NULL immediately, indicating that no valid option was found.
    - If the `only` flag is set, it calls [`options_get_only`](#options_get_only) to retrieve the option directly from the current options structure.
    - If the `only` flag is not set, it calls [`options_get`](#options_get) to search for the option in the current options structure and its parent structures.
    - Finally, it frees the allocated memory for the parsed name and returns the retrieved `options_entry`.
- **Output**:
    - Returns a pointer to the `options_entry` corresponding to the parsed option name, or NULL if the option could not be found.
- **Functions called**:
    - [`options_parse`](#options_parse)
    - [`options_get_only`](#options_get_only)
    - [`options_get`](#options_get)


---
### options_match<!-- {{#callable:options_match}} -->
The `options_match` function attempts to match a given string against a predefined options table and returns the corresponding option name.
- **Inputs**:
    - `s`: A constant character pointer representing the string to match against the options.
    - `idx`: A pointer to an integer that will hold the index parsed from the input string.
    - `ambiguous`: A pointer to an integer that indicates whether the match is ambiguous.
- **Control Flow**:
    - The function first calls [`options_parse`](#options_parse) to parse the input string `s` and retrieve a parsed string.
    - If the parsed string is NULL, the function returns NULL, indicating no match.
    - If the parsed string starts with '@', it sets `*ambiguous` to 0 and returns the parsed string directly.
    - The function then maps the parsed string to an option name using [`options_map_name`](#options_map_name).
    - It iterates through the `options_table` to find a matching option name.
    - If a match is found, it checks for ambiguity by looking for other options that match the beginning of the name.
    - If multiple matches are found, it sets `*ambiguous` to 1, frees the parsed string, and returns NULL.
    - If no matches are found, it sets `*ambiguous` to 0 and returns NULL.
    - If a single match is found, it duplicates the matched option name and returns it.
- **Output**:
    - The function returns a dynamically allocated string containing the matched option name, or NULL if no match is found or if the match is ambiguous.
- **Functions called**:
    - [`options_parse`](#options_parse)
    - [`options_map_name`](#options_map_name)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### options_match_get<!-- {{#callable:options_match_get}} -->
Retrieves an `options_entry` that matches a given string from a specified options structure.
- **Inputs**:
    - `oo`: A pointer to an `options` structure that contains the options to search.
    - `s`: A string to match against the option names.
    - `idx`: A pointer to an integer that will hold the index of the matched option if applicable.
    - `only`: An integer flag indicating whether to retrieve only the exact match.
    - `ambiguous`: A pointer to an integer that will be set to indicate if the match is ambiguous.
- **Control Flow**:
    - Calls [`options_match`](#options_match) to find a matching option name based on the input string `s`.
    - If no match is found (i.e., `name` is NULL), the function returns NULL.
    - Sets the `ambiguous` flag to 0 to indicate that the match is not ambiguous.
    - If the `only` flag is set, it calls [`options_get_only`](#options_get_only) to retrieve the exact match; otherwise, it calls [`options_get`](#options_get) to retrieve the match from the options structure.
    - Frees the allocated memory for `name` before returning the matched option entry.
- **Output**:
    - Returns a pointer to the `options_entry` that matches the input string, or NULL if no match is found.
- **Functions called**:
    - [`options_match`](#options_match)
    - [`options_get_only`](#options_get_only)
    - [`options_get`](#options_get)


---
### options_get_string<!-- {{#callable:options_get_string}} -->
Retrieves the string value of a specified option from the options structure.
- **Inputs**:
    - `oo`: A pointer to the `options` structure from which the option is to be retrieved.
    - `name`: A string representing the name of the option whose value is to be fetched.
- **Control Flow**:
    - Calls [`options_get`](#options_get) to retrieve the `options_entry` associated with the given `name` from the `options` structure.
    - If the retrieved `options_entry` is `NULL`, it triggers a fatal error indicating that the option is missing.
    - Checks if the retrieved option is of type string using the `OPTIONS_IS_STRING` macro; if not, it triggers a fatal error.
    - Returns the string value of the option from the `options_entry`.
- **Output**:
    - Returns a pointer to the string value of the specified option.
- **Functions called**:
    - [`options_get`](#options_get)


---
### options_get_number<!-- {{#callable:options_get_number}} -->
The `options_get_number` function retrieves a numeric option value from a given options structure.
- **Inputs**:
    - `oo`: A pointer to the `options` structure from which the numeric option is to be retrieved.
    - `name`: A string representing the name of the option to retrieve.
- **Control Flow**:
    - The function calls [`options_get`](#options_get) to find the `options_entry` associated with the provided `name` in the `options` structure.
    - If the `options_entry` is not found, it triggers a fatal error indicating the option is missing.
    - It then checks if the found option is of type number using the `OPTIONS_IS_NUMBER` macro.
    - If the option is not a number, it triggers a fatal error indicating the type mismatch.
    - Finally, it returns the numeric value of the option from the `options_entry`.
- **Output**:
    - Returns the numeric value of the specified option as a `long long` type.
- **Functions called**:
    - [`options_get`](#options_get)


---
### options_get_command<!-- {{#callable:options_get_command}} -->
The `options_get_command` function retrieves a command option from a given options structure.
- **Inputs**:
    - `oo`: A pointer to an `options` structure that contains the options from which to retrieve the command.
    - `name`: A string representing the name of the command option to retrieve.
- **Control Flow**:
    - The function calls [`options_get`](#options_get) to find the `options_entry` associated with the provided `name` in the `options` structure.
    - If the `options_entry` is not found (i.e., `o` is NULL), it triggers a fatal error indicating that the option is missing.
    - Next, it checks if the retrieved option is indeed a command using the `OPTIONS_IS_COMMAND` macro.
    - If the option is not a command, it triggers another fatal error indicating that the option is not a command.
    - Finally, if all checks pass, it returns the command list associated with the `options_entry`.
- **Output**:
    - The function returns a pointer to a `cmd_list` structure that represents the command associated with the specified option name.
- **Functions called**:
    - [`options_get`](#options_get)


---
### options_set_string<!-- {{#callable:options_set_string}} -->
Sets a string option in the options structure, optionally appending to an existing value.
- **Inputs**:
    - `oo`: A pointer to the `options` structure where the option is to be set.
    - `name`: A string representing the name of the option to set.
    - `append`: An integer flag indicating whether to append the new value to the existing value.
    - `fmt`: A format string for the new value, followed by a variable number of arguments.
- **Control Flow**:
    - Initializes a variable argument list and formats the new string value using [`xvasprintf`](xmalloc.c.driver.md#xvasprintf).
    - Retrieves the existing option entry using [`options_get_only`](#options_get_only).
    - If the option exists and `append` is true, concatenates the existing string with the new value, using a separator if defined.
    - If the option does not exist and the name starts with '@', it adds a new option entry.
    - If the option does not exist, it retrieves the default option entry from the parent options.
    - Checks if the option is of string type; if not, it raises a fatal error.
    - Frees the old string value and assigns the new value to the option entry.
- **Output**:
    - Returns a pointer to the `options_entry` structure representing the updated option.
- **Functions called**:
    - [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)
    - [`options_get_only`](#options_get_only)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`options_add`](#options_add)
    - [`options_default`](#options_default)
    - [`options_parent_table_entry`](#options_parent_table_entry)


---
### options_set_number<!-- {{#callable:options_set_number}} -->
Sets a numeric value for a specified option in the options structure.
- **Inputs**:
    - `oo`: A pointer to the `options` structure that contains the options.
    - `name`: A string representing the name of the option to set.
    - `value`: A long long integer representing the numeric value to assign to the option.
- **Control Flow**:
    - Checks if the first character of `name` is '@', and if so, calls `fatalx` to terminate the program with an error message.
    - Calls [`options_get_only`](#options_get_only) to retrieve the `options_entry` associated with `name` from the `options` structure.
    - If the `options_entry` is not found, it calls [`options_default`](#options_default) to get the default entry for the option based on its parent table entry.
    - If the `options_entry` is still NULL after this, the function returns NULL.
    - Checks if the retrieved `options_entry` is of type number using the `OPTIONS_IS_NUMBER` macro, and if not, calls `fatalx` to terminate with an error.
    - Sets the `number` field of the `options_entry` to the provided `value`.
- **Output**:
    - Returns a pointer to the `options_entry` that was modified, or NULL if the option could not be set.
- **Functions called**:
    - [`options_get_only`](#options_get_only)
    - [`options_default`](#options_default)
    - [`options_parent_table_entry`](#options_parent_table_entry)


---
### options_set_command<!-- {{#callable:options_set_command}} -->
Sets a command option in the options structure.
- **Inputs**:
    - `oo`: A pointer to the `options` structure where the command option is to be set.
    - `name`: A string representing the name of the command option to be set.
    - `value`: A pointer to a `cmd_list` structure representing the command list to be assigned to the option.
- **Control Flow**:
    - Checks if the `name` starts with '@', and if so, calls `fatalx` to terminate with an error message.
    - Retrieves the options entry associated with `name` using [`options_get_only`](#options_get_only).
    - If the entry is not found, it attempts to get the default entry from the parent options using [`options_default`](#options_default).
    - Checks if the retrieved options entry is a command type using `OPTIONS_IS_COMMAND`, and if not, calls `fatalx` to terminate with an error message.
    - If the command list in the options entry is not NULL, it frees the existing command list using [`cmd_list_free`](cmd.c.driver.md#cmd_list_free).
    - Assigns the new command list `value` to the options entry and returns the updated entry.
- **Output**:
    - Returns a pointer to the updated `options_entry` structure that now contains the new command list, or NULL if the entry could not be created.
- **Functions called**:
    - [`options_get_only`](#options_get_only)
    - [`options_default`](#options_default)
    - [`options_parent_table_entry`](#options_parent_table_entry)
    - [`cmd_list_free`](cmd.c.driver.md#cmd_list_free)


---
### options_scope_from_name<!-- {{#callable:options_scope_from_name}} -->
The `options_scope_from_name` function determines the scope of options based on a given name and updates the provided options pointer.
- **Inputs**:
    - `args`: A pointer to a struct `args` that contains command-line arguments.
    - `window`: An integer representing the window identifier.
    - `name`: A string representing the name of the option whose scope is to be determined.
    - `fs`: A pointer to a struct `cmd_find_state` that holds the current state of command finding.
    - `oo`: A pointer to a pointer of struct `options` where the resulting options will be stored.
    - `cause`: A pointer to a string that will hold an error message if the operation fails.
- **Control Flow**:
    - Checks if the first character of `name` is '@', and if so, calls [`options_scope_from_flags`](#options_scope_from_flags) to handle it.
    - Iterates through the `options_table` to find the corresponding option entry for the provided `name`.
    - If the option is not found, it sets an error message in `cause` and returns `OPTIONS_TABLE_NONE`.
    - Based on the scope of the found option entry, it updates the `oo` pointer to point to the appropriate options structure.
    - Handles different cases for server, session, window, and pane scopes, checking for the existence of the respective entities.
- **Output**:
    - Returns an integer representing the scope of the option, which can be one of the predefined scope constants such as `OPTIONS_TABLE_SERVER`, `OPTIONS_TABLE_SESSION`, `OPTIONS_TABLE_WINDOW`, or `OPTIONS_TABLE_PANE`.
- **Functions called**:
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`options_scope_from_flags`](#options_scope_from_flags)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`args_has`](arguments.c.driver.md#args_has)


---
### options_scope_from_flags<!-- {{#callable:options_scope_from_flags}} -->
The `options_scope_from_flags` function determines the scope of options based on command-line flags and updates the provided options pointer.
- **Inputs**:
    - `args`: A pointer to a `struct args` that contains command-line arguments.
    - `window`: An integer indicating whether the context is a window.
    - `fs`: A pointer to a `struct cmd_find_state` that holds the current state of command finding.
    - `oo`: A pointer to a pointer of `struct options` where the resulting options will be stored.
    - `cause`: A pointer to a character pointer that will hold error messages if any issues occur.
- **Control Flow**:
    - Checks if the 's' flag is present in `args`, if so, sets `*oo` to `global_options` and returns `OPTIONS_TABLE_SERVER`.
    - If the 'p' flag is present, it checks if the current pane (`wp`) is valid; if not, it sets an error message in `cause` and returns `OPTIONS_TABLE_NONE`.
    - If the `window` parameter is true or the 'w' flag is present, it checks for the 'g' flag to set `*oo` to `global_w_options` or validates the current window (`wl`) and sets `*oo` accordingly.
    - If none of the above conditions are met, it checks for the 'g' flag to set `*oo` to `global_s_options` or validates the current session (`s`) and sets `*oo` accordingly.
- **Output**:
    - Returns an integer representing the options scope: `OPTIONS_TABLE_SERVER`, `OPTIONS_TABLE_PANE`, `OPTIONS_TABLE_WINDOW`, `OPTIONS_TABLE_SESSION`, or `OPTIONS_TABLE_NONE` if an error occurs.
- **Functions called**:
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### options_string_to_style<!-- {{#callable:options_string_to_style}} -->
Converts a string option to a `style` structure.
- **Inputs**:
    - `oo`: A pointer to the `options` structure containing the options.
    - `name`: A string representing the name of the option to retrieve.
    - `ft`: A pointer to a `format_tree` structure used for expanding the option's value.
- **Control Flow**:
    - Retrieve the option entry corresponding to `name` from the `options` structure.
    - Check if the option entry is valid and of string type; if not, return NULL.
    - If the style is cached, return the cached style.
    - Log the string value of the option for debugging purposes.
    - Set the default style based on `grid_default_cell`.
    - Determine if the string contains format placeholders; if not, cache the style.
    - If a format tree is provided and the style is not cached, expand the string using [`format_expand`](format.c.driver.md#format_expand).
    - Parse the expanded string into a style using [`style_parse`](style.c.driver.md#style_parse).
    - If parsing fails, free the expanded string and return NULL.
    - If no format tree is provided, parse the original string directly into a style.
    - Return a pointer to the populated style structure.
- **Output**:
    - Returns a pointer to the `style` structure corresponding to the option, or NULL if an error occurs.
- **Functions called**:
    - [`options_get`](#options_get)
    - [`style_set`](style.c.driver.md#style_set)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`style_parse`](style.c.driver.md#style_parse)


---
### options_from_string_check<!-- {{#callable:options_from_string_check}} -->
The `options_from_string_check` function validates a given string value against specific criteria defined in an options table entry.
- **Inputs**:
    - `oe`: A pointer to an `options_table_entry` structure that defines the criteria for validation.
    - `value`: A string representing the value to be validated.
    - `cause`: A pointer to a character pointer that will hold an error message if validation fails.
- **Control Flow**:
    - The function first checks if the `oe` pointer is NULL, returning 0 if it is.
    - It then checks if the name of the option is 'default-shell' and if the provided value is a valid shell using the [`checkshell`](tmux.c.driver.md#checkshell) function; if not, it sets an error message and returns -1.
    - Next, it checks if the `oe` has a pattern and if the value matches this pattern using `fnmatch`; if it doesn't match, it sets an error message and returns -1.
    - Finally, if the option is of style type and the value does not contain a specific substring, it attempts to parse the style using [`style_parse`](style.c.driver.md#style_parse); if parsing fails, it sets an error message and returns -1.
    - If all checks pass, the function returns 0, indicating successful validation.
- **Output**:
    - The function returns 0 if the value is valid according to the criteria, or -1 if it is invalid, with an error message stored in `cause`.
- **Functions called**:
    - [`checkshell`](tmux.c.driver.md#checkshell)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`style_parse`](style.c.driver.md#style_parse)


---
### options_from_string_flag<!-- {{#callable:options_from_string_flag}} -->
The `options_from_string_flag` function sets a flag option based on a string value.
- **Inputs**:
    - `oo`: A pointer to the `options` structure that holds the options.
    - `name`: A string representing the name of the option to set.
    - `value`: A string representing the value to set the option to, which can be '1', '0', 'on', 'off', 'yes', 'no', or NULL.
    - `cause`: A pointer to a string that will hold an error message if the function fails.
- **Control Flow**:
    - The function first checks if the `value` is NULL or an empty string; if so, it sets the flag to the opposite of the current value.
    - If `value` is '1', 'on', or 'yes', it sets the flag to 1.
    - If `value` is '0', 'off', or 'no', it sets the flag to 0.
    - If `value` does not match any of the expected strings, it generates an error message and returns -1.
    - If the flag is successfully set, the function returns 0.
- **Output**:
    - The function returns 0 on success or -1 on failure, with an error message stored in `cause` if applicable.
- **Functions called**:
    - [`options_get_number`](#options_get_number)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`options_set_number`](#options_set_number)


---
### options_find_choice<!-- {{#callable:options_find_choice}} -->
The `options_find_choice` function searches for a specified value in a list of choices and returns its index.
- **Inputs**:
    - `oe`: A pointer to an `options_table_entry` structure that contains an array of choices.
    - `value`: A string representing the value to search for within the choices.
    - `cause`: A pointer to a string that will hold an error message if the value is not found.
- **Control Flow**:
    - Iterates through the `choices` array in the `options_table_entry` structure.
    - Compares each choice with the provided `value` using `strcmp`.
    - If a match is found, the index of the choice is stored in the `choice` variable.
    - If no match is found after the loop, an error message is generated and -1 is returned.
    - If a match is found, the index of the choice is returned.
- **Output**:
    - Returns the index of the found choice or -1 if the choice is not found, with an error message stored in `cause`.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### options_from_string_choice<!-- {{#callable:options_from_string_choice}} -->
The `options_from_string_choice` function sets an option's value based on a string representation of a choice.
- **Inputs**:
    - `oe`: A pointer to the `options_table_entry` structure that defines the option's properties.
    - `oo`: A pointer to the `options` structure where the option value will be set.
    - `name`: A string representing the name of the option to be set.
    - `value`: A string representing the choice value to be set; if NULL, the current value is retrieved.
    - `cause`: A pointer to a string that will hold an error message if an error occurs.
- **Control Flow**:
    - If `value` is NULL, the function retrieves the current choice value using [`options_get_number`](#options_get_number).
    - If the retrieved choice is less than 2, it toggles the choice value.
    - If `value` is not NULL, it attempts to find the corresponding choice index using [`options_find_choice`](#options_find_choice).
    - If [`options_find_choice`](#options_find_choice) returns a negative value, indicating an error, the function returns -1.
    - Finally, it sets the option's number value using [`options_set_number`](#options_set_number) and returns 0.
- **Output**:
    - The function returns 0 on success, or -1 if an error occurs while finding the choice.
- **Functions called**:
    - [`options_get_number`](#options_get_number)
    - [`options_find_choice`](#options_find_choice)
    - [`options_set_number`](#options_set_number)


---
### options_from_string<!-- {{#callable:options_from_string}} -->
The `options_from_string` function parses a string value and updates an options structure based on the specified option type.
- **Inputs**:
    - `oo`: A pointer to the `options` structure that will be modified.
    - `oe`: A pointer to the `options_table_entry` structure that defines the type and constraints of the option.
    - `name`: A string representing the name of the option to be set.
    - `value`: A string containing the value to be parsed and assigned to the option.
    - `append`: An integer flag indicating whether to append the value to an existing string option.
    - `cause`: A pointer to a string that will hold an error message if the operation fails.
- **Control Flow**:
    - The function first checks if the `oe` pointer is not NULL to determine the option type; if it is NULL, it validates that the `name` starts with '@'.
    - Based on the option type, it enters a switch statement to handle different types of options: string, number, key, colour, flag, choice, and command.
    - For string options, it duplicates the current value, sets the new value, and checks if the new value is valid; if not, it reverts to the old value.
    - For number options, it converts the string to a number and checks if it falls within the specified range.
    - For key options, it looks up the key code from the string value.
    - For colour options, it converts the string to a colour value.
    - For flag options, it toggles the flag based on the provided value.
    - For choice options, it finds the corresponding choice index from the provided value.
    - For command options, it parses the command string and sets it in the options structure.
    - If any operation fails, it sets an error message in `cause` and returns -1; otherwise, it returns 0 on success.
- **Output**:
    - The function returns 0 on success or -1 on failure, with an error message stored in `cause` if applicable.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`options_get_string`](#options_get_string)
    - [`options_set_string`](#options_set_string)
    - [`options_from_string_check`](#options_from_string_check)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
    - [`options_set_number`](#options_set_number)
    - [`key_string_lookup_string`](key-string.c.driver.md#key_string_lookup_string)
    - [`colour_fromstring`](colour.c.driver.md#colour_fromstring)
    - [`options_from_string_flag`](#options_from_string_flag)
    - [`options_from_string_choice`](#options_from_string_choice)
    - [`options_set_command`](#options_set_command)


---
### options_push_changes<!-- {{#callable:options_push_changes}} -->
The `options_push_changes` function updates various settings in a terminal multiplexer based on the provided option name.
- **Inputs**:
    - `name`: A string representing the name of the option whose changes are to be pushed.
- **Control Flow**:
    - Logs the function call with the provided option name.
    - Checks the option name against various predefined cases using `strcmp`.
    - For each case, iterates over relevant data structures (like windows or panes) to apply changes based on the option.
    - Updates the status of sessions and redraws clients as necessary.
- **Output**:
    - The function does not return a value; it modifies the state of the application based on the option changes.
- **Functions called**:
    - [`options_get_number`](#options_get_number)
    - [`window_pane_default_cursor`](window.c.driver.md#window_pane_default_cursor)
    - [`window_set_fill_character`](window.c.driver.md#window_set_fill_character)
    - [`server_client_set_key_table`](server-client.c.driver.md#server_client_set_key_table)
    - [`tty_keys_build`](tty-keys.c.driver.md#tty_keys_build)
    - [`status_timer_start_all`](status.c.driver.md#status_timer_start_all)
    - [`alerts_reset_all`](alerts.c.driver.md#alerts_reset_all)
    - [`colour_palette_from_option`](colour.c.driver.md#colour_palette_from_option)
    - [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes)
    - [`style_set_scrollbar_style_from_option`](style.c.driver.md#style_set_scrollbar_style_from_option)
    - [`utf8_update_width_cache`](utf8.c.driver.md#utf8_update_width_cache)
    - [`input_set_buffer_size`](input.c.driver.md#input_set_buffer_size)
    - [`status_update_cache`](status.c.driver.md#status_update_cache)
    - [`recalculate_sizes`](resize.c.driver.md#recalculate_sizes)
    - [`server_redraw_client`](server-fn.c.driver.md#server_redraw_client)


---
### options_remove_or_default<!-- {{#callable:options_remove_or_default}} -->
Removes an option or sets it to its default value based on the provided index.
- **Inputs**:
    - `o`: A pointer to the `options_entry` structure representing the option to be removed or reset.
    - `idx`: An integer index indicating the specific array element to modify; if -1, the entire option is affected.
    - `cause`: A pointer to a character pointer that can be used to return an error message if the operation fails.
- **Control Flow**:
    - Checks if the index `idx` is -1, indicating that the entire option should be removed or reset to default.
    - If `idx` is -1 and the option has a `tableentry`, it checks if the owner of the option is one of the global options.
    - If the owner is a global option, it calls [`options_default`](#options_default) to reset the option to its default value.
    - If the owner is not a global option, it calls [`options_remove`](#options_remove) to remove the option entirely.
    - If `idx` is not -1, it attempts to set the specified index of the option's array to NULL using [`options_array_set`](#options_array_set).
    - If [`options_array_set`](#options_array_set) fails, it returns -1; otherwise, it returns 0 to indicate success.
- **Output**:
    - Returns 0 on success, or -1 if an error occurs during the operation.
- **Functions called**:
    - [`options_default`](#options_default)
    - [`options_remove`](#options_remove)
    - [`options_array_set`](#options_array_set)


