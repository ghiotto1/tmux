# Purpose
This C source code file is part of the tmux project, specifically handling the management of options within the application. The code defines a system for managing options, which are stored in a red-black tree structure. Each option has a name, type, and value, and can be organized hierarchically with parent-child relationships. The file provides functions to create, retrieve, modify, and delete options, as well as to convert option values to and from strings. It supports various types of options, including strings, numbers, commands, and arrays, and includes mechanisms for handling default values and validating option values against specified patterns or constraints.

The code is structured around several key data structures, such as `options`, `options_entry`, and [`options_array_item`](#options_array_item), which represent the options tree, individual options, and array items, respectively. The file includes functions for comparing, adding, and removing options, as well as for parsing and setting option values. It also includes utility functions for handling specific option types, such as flags and choices, and for managing the scope of options within different contexts, such as server, session, window, and pane. The file is designed to be integrated into the larger tmux codebase, providing a robust and flexible system for managing configuration options within the application.
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
- **Type**: `function pointer`
- **Description**: `options_add` is a static function pointer that returns a pointer to an `options_entry` structure. It takes two parameters: a pointer to an `options` structure and a constant character pointer. This function is used to add a new option entry to the options tree.
- **Use**: This function is used to add a new option entry to the options tree within the `options` structure.


# Data Structures

---
### options_array_item
- **Type**: `struct`
- **Members**:
    - `index`: An unsigned integer representing the index of the item in the array.
    - `value`: A union representing the value of the option, which can be of various types.
    - `entry`: A red-black tree entry used to organize the items in a red-black tree.
- **Description**: The `options_array_item` structure is used to represent an individual item within an options array in the tmux codebase. Each item has an index to identify its position, a value that can be of different types as defined by the `options_value` union, and an entry for integration into a red-black tree, which allows for efficient searching, insertion, and deletion operations. This structure is part of the mechanism for handling options in tmux, where options can be stored in arrays and managed using a red-black tree for optimal performance.


---
### options_entry
- **Type**: `struct`
- **Members**:
    - `owner`: A pointer to the options structure that owns this entry.
    - `name`: A constant character pointer representing the name of the option.
    - `tableentry`: A pointer to the options_table_entry structure that defines the option's properties.
    - `value`: A union options_value that holds the value of the option.
    - `cached`: An integer indicating whether the style is cached.
    - `style`: A struct style that holds styling information for the option.
    - `entry`: A red-black tree entry for managing the options_entry in a tree structure.
- **Description**: The `options_entry` structure represents an individual option within a configuration system, typically used in applications like tmux. It contains metadata about the option, such as its name and type, as well as its current value. The structure is designed to be part of a red-black tree, allowing for efficient storage and retrieval of options. The `owner` field links the entry to its parent options structure, while the `tableentry` provides a reference to the option's definition in a table of possible options. The `value` field stores the actual value of the option, which can be of various types as defined by the `options_value` union. The `style` field is used for options that involve styling, and the `cached` field indicates if the style information is cached for performance reasons.


---
### options
- **Type**: `struct`
- **Members**:
    - `tree`: A red-black tree that stores options entries.
    - `parent`: A pointer to the parent options structure, allowing for hierarchical option inheritance.
- **Description**: The `options` structure is designed to manage a collection of configuration options, each identified by a name, type, and value, organized in a red-black tree for efficient retrieval and manipulation. It supports hierarchical option management by allowing each `options` instance to have a parent, enabling inheritance of options from a parent structure. This is particularly useful in scenarios where options need to be overridden or extended in a nested configuration context.


# Functions

---
### options_array_cmp<!-- {{#callable:options_array_cmp}} -->
Compares two `options_array_item` structures based on their index values.
- **Inputs**:
    - `a1`: Pointer to the first `options_array_item` structure to compare.
    - `a2`: Pointer to the second `options_array_item` structure to compare.
- **Control Flow**:
    - The function first checks if the index of `a1` is less than that of `a2`, returning -1 if true.
    - If the index of `a1` is greater than that of `a2`, it returns 1.
    - If both indices are equal, it returns 0.
- **Output**:
    - Returns an integer: -1 if `a1` is less than `a2`, 1 if greater, and 0 if they are equal.


---
### options_cmp<!-- {{#callable:options_cmp}} -->
Compares two `options_entry` structures based on their `name` field.
- **Inputs**:
    - `lhs`: A pointer to the first `options_entry` structure to compare.
    - `rhs`: A pointer to the second `options_entry` structure to compare.
- **Control Flow**:
    - The function uses `strcmp` to compare the `name` fields of the two `options_entry` structures.
    - It returns the result of the comparison, which indicates the lexicographical order of the names.
- **Output**:
    - An integer value: negative if `lhs->name` is less than `rhs->name`, positive if greater, and zero if they are equal.


---
### options_map_name<!-- {{#callable:options_map_name}} -->
The `options_map_name` function maps a given name to its corresponding option name using a predefined mapping structure.
- **Inputs**:
    - `name`: A constant character pointer representing the name to be mapped.
- **Control Flow**:
    - Iterates through an array of `options_name_map` structures until it finds a match for the input `name`.
    - Compares each `from` field in the mapping structure with the input `name` using `strcmp`.
    - If a match is found, it returns the corresponding `to` field from the mapping structure.
    - If no match is found by the end of the array, it returns the original `name`.
- **Output**:
    - Returns a constant character pointer to the mapped name if found; otherwise, it returns the original input name.


---
### options_parent_table_entry<!-- {{#callable:options_parent_table_entry}} -->
Retrieves the table entry of a specified option from the parent options structure.
- **Inputs**:
    - `oo`: A pointer to the `options` structure from which the parent options are accessed.
    - `s`: A string representing the name of the option whose table entry is to be retrieved.
- **Control Flow**:
    - Checks if the `parent` of the `options` structure is NULL; if so, it triggers a fatal error indicating that there are no parent options.
    - Calls the [`options_get`](#options_get) function to retrieve the `options_entry` associated with the specified option name from the parent options.
    - If the retrieved `options_entry` is NULL, it triggers a fatal error indicating that the specified option is not found in the parent options.
    - Returns the `tableentry` from the found `options_entry`.
- **Output**:
    - Returns a pointer to the `options_table_entry` associated with the specified option name from the parent options.
- **Functions called**:
    - [`options_get`](#options_get)


---
### options_value_free<!-- {{#callable:options_value_free}} -->
The `options_value_free` function frees the memory associated with the value of an options entry based on its type.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure representing the options entry whose value is to be freed.
    - `ov`: A pointer to a `union options_value` that holds the value associated with the options entry.
- **Control Flow**:
    - The function first checks if the type of the options entry `o` is a string using the `OPTIONS_IS_STRING` macro.
    - If it is a string, it frees the memory allocated for the string value in `ov->string`.
    - Next, it checks if the type of the options entry is a command using the `OPTIONS_IS_COMMAND` macro and if `ov->cmdlist` is not NULL.
    - If both conditions are met, it calls `cmd_list_free` to free the command list associated with the options entry.
- **Output**:
    - The function does not return a value; it performs memory deallocation to prevent memory leaks.


---
### options_value_to_string<!-- {{#callable:options_value_to_string}} -->
Converts an `options_entry` value to its string representation based on its type.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure that contains the option's metadata including its type.
    - `ov`: A pointer to a union `options_value` that holds the actual value of the option.
    - `numeric`: An integer flag indicating whether to return numeric representations for certain types.
- **Control Flow**:
    - Checks if the option is a command using `OPTIONS_IS_COMMAND` macro; if true, calls `cmd_list_print` to get its string representation.
    - Checks if the option is a number using `OPTIONS_IS_NUMBER` macro; if true, it further checks the specific type of number and formats it accordingly.
    - For `OPTIONS_TABLE_NUMBER`, it formats the number as a long long integer string.
    - For `OPTIONS_TABLE_KEY`, it looks up the key string representation.
    - For `OPTIONS_TABLE_COLOUR`, it converts the number to a color string.
    - For `OPTIONS_TABLE_FLAG`, it checks the `numeric` flag to decide whether to return 'on'/'off' or the numeric value.
    - For `OPTIONS_TABLE_CHOICE`, it retrieves the corresponding choice string from the options entry.
    - If the option is a string, it duplicates the string value directly.
    - If none of the conditions are met, it returns an empty string.
- **Output**:
    - Returns a dynamically allocated string representation of the option's value, or an empty string if the option type is unrecognized.


---
### options_create<!-- {{#callable:options_create}} -->
Creates a new `options` structure with an optional parent.
- **Inputs**:
    - `parent`: A pointer to an existing `options` structure that serves as the parent of the new options.
- **Control Flow**:
    - Allocates memory for a new `options` structure using `xcalloc`, initializing it to zero.
    - Initializes the red-black tree within the new `options` structure using `RB_INIT`.
    - Sets the `parent` field of the new `options` structure to the provided `parent` argument.
    - Returns a pointer to the newly created `options` structure.
- **Output**:
    - Returns a pointer to the newly created `options` structure.


---
### options_free<!-- {{#callable:options_free}} -->
The `options_free` function deallocates memory associated with an `options` structure and its entries.
- **Inputs**:
    - `oo`: A pointer to the `options` structure that needs to be freed.
- **Control Flow**:
    - Iterates over each `options_entry` in the red-black tree of the `options` structure using `RB_FOREACH_SAFE`.
    - For each entry, it calls the [`options_remove`](#options_remove) function to remove and free the entry.
    - Finally, it frees the memory allocated for the `options` structure itself.
- **Output**:
    - The function does not return a value; it performs memory deallocation.
- **Functions called**:
    - [`options_remove`](#options_remove)


---
### options_get_parent<!-- {{#callable:options_get_parent}} -->
The `options_get_parent` function retrieves the parent options structure of a given options structure.
- **Inputs**:
    - `oo`: A pointer to an `options` structure whose parent is to be retrieved.
- **Control Flow**:
    - The function directly accesses the `parent` member of the `options` structure pointed to by `oo`.
    - It returns the value of the `parent` member, which is also a pointer to another `options` structure.
- **Output**:
    - The function returns a pointer to the parent `options` structure, or NULL if there is no parent.


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
The `options_get_only` function retrieves an `options_entry` from a red-black tree based on a specified name.
- **Inputs**:
    - `oo`: A pointer to an `options` structure that contains the red-black tree of options.
    - `name`: A string representing the name of the option to retrieve.
- **Control Flow**:
    - An `options_entry` structure is initialized with the provided `name`.
    - The function attempts to find the entry in the red-black tree using `RB_FIND`.
    - If the entry is not found, the function maps the name using [`options_map_name`](#options_map_name) and tries to find it again.
    - If found, the function returns the corresponding `options_entry`; otherwise, it returns NULL.
- **Output**:
    - Returns a pointer to the `options_entry` if found, or NULL if not found.
- **Functions called**:
    - [`options_map_name`](#options_map_name)


---
### options_get<!-- {{#callable:options_get}} -->
The `options_get` function retrieves an `options_entry` by name from a given `options` structure, searching through parent options if necessary.
- **Inputs**:
    - `oo`: A pointer to an `options` structure from which to retrieve the option.
    - `name`: A string representing the name of the option to retrieve.
- **Control Flow**:
    - The function first attempts to find the option using [`options_get_only`](#options_get_only) with the provided `options` structure.
    - If the option is not found (i.e., `o` is NULL`), it traverses up the parent options structure until it either finds the option or reaches the top (NULL parent).
    - The loop continues until either the option is found or there are no more parent options to check.
- **Output**:
    - Returns a pointer to the `options_entry` if found, or NULL if the option does not exist in the given options or any of its parents.
- **Functions called**:
    - [`options_get_only`](#options_get_only)


---
### options_empty<!-- {{#callable:options_empty}} -->
The `options_empty` function initializes a new `options_entry` with a specified name and associates it with a given options table entry.
- **Inputs**:
    - `oo`: A pointer to an `options` structure representing the current options context.
    - `oe`: A pointer to an `options_table_entry` structure that contains metadata about the option being added.
- **Control Flow**:
    - Calls the [`options_add`](#options_add) function to create a new `options_entry` with the name from the `options_table_entry`.
    - Assigns the `tableentry` field of the newly created `options_entry` to point to the provided `options_table_entry`.
    - Checks if the `flags` field of the `options_table_entry` indicates that it is an array, and if so, initializes the `value.array` field of the `options_entry`.
- **Output**:
    - Returns a pointer to the newly created `options_entry`.
- **Functions called**:
    - [`options_add`](#options_add)


---
### options_default<!-- {{#callable:options_default}} -->
The `options_default` function initializes an `options_entry` structure with default values based on the provided options table entry.
- **Inputs**:
    - `oo`: A pointer to an `options` structure that represents the current options context.
    - `oe`: A pointer to an `options_table_entry` structure that contains the definition of the option, including its type and default values.
- **Control Flow**:
    - Calls [`options_empty`](#options_empty) to create an empty `options_entry` and associates it with the provided options table entry.
    - Checks if the option is an array by evaluating the `OPTIONS_TABLE_IS_ARRAY` flag in `oe->flags`.
    - If it is an array and `oe->default_arr` is NULL, it assigns a default string value using [`options_array_assign`](#options_array_assign).
    - If it is an array and `oe->default_arr` is not NULL, it iterates through the default array and sets each value using [`options_array_set`](#options_array_set).
    - If the option is not an array, it checks the type of the option (`OPTIONS_TABLE_STRING`, `OPTIONS_TABLE_COMMAND`, or others) and assigns the appropriate default value to `ov`.
- **Output**:
    - Returns a pointer to the initialized `options_entry` structure containing the default values.
- **Functions called**:
    - [`options_empty`](#options_empty)
    - [`options_array_assign`](#options_array_assign)
    - [`options_array_set`](#options_array_set)


---
### options_default_to_string<!-- {{#callable:options_default_to_string}} -->
Converts the default value of an options table entry to a string representation.
- **Inputs**:
    - `oe`: A pointer to a `struct options_table_entry` that contains the type and default value of the option.
- **Control Flow**:
    - The function begins by checking the `type` of the option represented by `oe`.
    - Depending on the `type`, it handles the conversion of the default value to a string in different ways.
    - For string and command types, it duplicates the `default_str` field.
    - For number types, it formats the number as a string using `xasprintf`.
    - For key and color types, it looks up the corresponding string representation using helper functions.
    - For flag types, it converts the boolean value to 'on' or 'off'.
    - For choice types, it retrieves the corresponding choice string from the `choices` array.
    - If an unknown type is encountered, it triggers a fatal error.
    - Finally, it returns the resulting string.
- **Output**:
    - Returns a pointer to a dynamically allocated string that represents the default value of the option. The caller is responsible for freeing this string.


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


---
### options_remove<!-- {{#callable:options_remove}} -->
The `options_remove` function deallocates and removes an `options_entry` from its owner options tree.
- **Inputs**:
    - `o`: A pointer to the `options_entry` structure that is to be removed.
- **Control Flow**:
    - The function retrieves the owner of the `options_entry` from the `owner` field.
    - It checks if the `options_entry` is an array using the `OPTIONS_IS_ARRAY` macro.
    - If it is an array, it calls [`options_array_clear`](#options_array_clear) to clear the array.
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
    - Returns a pointer to the `options` structure that is the owner of the specified `options_entry`.


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
    - A temporary `options_array_item` structure `a` is created and its index is set to the provided `idx`.
    - The function then calls `RB_FIND` to search for the item in the red-black tree associated with `o->value.array` using the temporary structure `a`.
- **Output**:
    - Returns a pointer to the found `options_array_item` if it exists, or NULL if no item with the specified index is found.


---
### options_array_new<!-- {{#callable:options_array_new}} -->
Creates a new `options_array_item` and inserts it into the options entry's array.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure that represents the options entry to which the new array item will be added.
    - `idx`: An unsigned integer representing the index of the new array item.
- **Control Flow**:
    - Allocates memory for a new `options_array_item` using `xcalloc`, initializing it to zero.
    - Sets the `index` field of the newly created item to the provided `idx` value.
    - Inserts the new item into the red-black tree associated with the `options_entry`'s array using `RB_INSERT`.
    - Returns a pointer to the newly created `options_array_item`.
- **Output**:
    - Returns a pointer to the newly created `options_array_item`.


---
### options_array_free<!-- {{#callable:options_array_free}} -->
The `options_array_free` function deallocates memory associated with an `options_array_item` and removes it from the corresponding options entry.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure that represents the options entry containing the array item.
    - `a`: A pointer to an `options_array_item` structure that is to be freed.
- **Control Flow**:
    - Calls [`options_value_free`](#options_value_free) to free the value associated with the array item `a`.
    - Removes the array item `a` from the red-black tree associated with the options entry `o` using `RB_REMOVE`.
    - Frees the memory allocated for the array item `a`.
- **Output**:
    - This function does not return a value; it performs memory deallocation and structural updates.
- **Functions called**:
    - [`options_value_free`](#options_value_free)


---
### options_array_clear<!-- {{#callable:options_array_clear}} -->
The `options_array_clear` function frees all items in an options array.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure that represents the options array to be cleared.
- **Control Flow**:
    - Checks if the provided `options_entry` is an array using the `OPTIONS_IS_ARRAY` macro.
    - If it is not an array, the function returns immediately without performing any action.
    - Iterates over each item in the options array using `RB_FOREACH_SAFE`, which allows safe removal of items during iteration.
    - For each item, it calls the [`options_array_free`](#options_array_free) function to free the memory associated with that item.
- **Output**:
    - The function does not return a value; it modifies the state of the options array by freeing its items.
- **Functions called**:
    - [`options_array_free`](#options_array_free)


---
### options_array_get<!-- {{#callable:options_array_get}} -->
Retrieves the value of an item from an options array.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure representing the options array.
    - `idx`: An unsigned integer representing the index of the item to retrieve from the options array.
- **Control Flow**:
    - Checks if the `options_entry` pointed to by `o` is an array using the `OPTIONS_IS_ARRAY` macro.
    - If `o` is not an array, the function returns NULL.
    - Calls the [`options_array_item`](#options_array_item) function to retrieve the array item at the specified index `idx`.
    - If the item is not found (i.e., `a` is NULL), the function returns NULL.
    - If the item is found, the function returns a pointer to the value of the found item.
- **Output**:
    - Returns a pointer to the `options_value` of the item at the specified index in the options array, or NULL if the array is not valid or the index is out of bounds.
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
    - `cause`: A pointer to a string that will hold an error message if an error occurs.
- **Control Flow**:
    - Checks if the `options_entry` is an array; if not, sets an error message and returns -1.
    - If `value` is NULL, it frees the array item at the specified index and returns 0.
    - If the option is a command, it parses the command string and handles errors accordingly.
    - If the option is a string, it either appends the new value to the existing string or sets it directly.
    - If the option type is color, it converts the string to a color number and handles errors.
    - If none of the conditions match, it sets an error message indicating the wrong array type and returns -1.
- **Output**:
    - Returns 0 on success, -1 on failure with an error message set in `cause` if applicable.
- **Functions called**:
    - [`options_array_item`](#options_array_item)
    - [`options_array_free`](#options_array_free)
    - [`options_array_new`](#options_array_new)
    - [`options_value_free`](#options_value_free)


---
### options_array_assign<!-- {{#callable:options_array_assign}} -->
The `options_array_assign` function assigns values to an options array from a given string, splitting the string based on a specified separator.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure representing the options array to be modified.
    - `s`: A string containing the values to be assigned to the options array.
    - `cause`: A pointer to a character pointer that can be used to return an error message if the assignment fails.
- **Control Flow**:
    - The function retrieves the separator from the `options_entry` structure; if none is provided, it defaults to a space or comma.
    - If the separator is an empty string, it checks if the input string `s` is empty and returns 0 if so.
    - If `s` is not empty, it duplicates the string and begins to split it using the specified separator.
    - For each non-empty substring obtained from the split, it finds the next available index in the options array and assigns the substring to that index using [`options_array_set`](#options_array_set).
    - If any assignment fails, it frees the duplicated string and returns -1.
    - Finally, it frees the duplicated string and returns 0 to indicate success.
- **Output**:
    - The function returns 0 on success, -1 if an error occurs during assignment, or 0 if the input string is empty.
- **Functions called**:
    - [`options_array_item`](#options_array_item)
    - [`options_array_set`](#options_array_set)


---
### options_array_first<!-- {{#callable:options_array_first}} -->
The `options_array_first` function retrieves the first item from an options array.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure that represents the options array.
- **Control Flow**:
    - The function first checks if the provided `options_entry` is an array using the `OPTIONS_IS_ARRAY` macro.
    - If the check fails, it returns `NULL` indicating that the entry is not an array.
    - If the check passes, it retrieves the first item in the array using `RB_MIN` on the red-black tree associated with the array.
- **Output**:
    - The function returns a pointer to the first `options_array_item` in the array if it exists, or `NULL` if the entry is not an array.


---
### options_array_next<!-- {{#callable:options_array_next}} -->
The `options_array_next` function retrieves the next item in a red-black tree of options array items.
- **Inputs**:
    - `a`: A pointer to the current `options_array_item` whose successor is to be found.
- **Control Flow**:
    - The function calls `RB_NEXT` macro to find the next item in the red-black tree.
    - It passes the `options_array` type, the address of the array associated with the current item, and the current item itself.
- **Output**:
    - Returns a pointer to the next `options_array_item` in the tree, or NULL if there is no next item.


---
### options_array_item_index<!-- {{#callable:options_array_item_index}} -->
The `options_array_item_index` function retrieves the index of a given `options_array_item`.
- **Inputs**:
    - `a`: A pointer to an `options_array_item` structure whose index is to be retrieved.
- **Control Flow**:
    - The function directly accesses the `index` field of the `options_array_item` structure pointed to by `a`.
    - It returns the value of the `index` field without any additional computation or checks.
- **Output**:
    - The function returns an unsigned integer representing the index of the specified `options_array_item`.


---
### options_array_item_value<!-- {{#callable:options_array_item_value}} -->
Returns a pointer to the `value` field of a given `options_array_item`.
- **Inputs**:
    - `a`: A pointer to an `options_array_item` structure.
- **Control Flow**:
    - The function directly accesses the `value` field of the `options_array_item` structure pointed to by `a`.
    - It returns the address of the `value` field, which is of type `union options_value`.
- **Output**:
    - A pointer to the `value` field of the `options_array_item`, which is of type `union options_value`.


---
### options_is_array<!-- {{#callable:options_is_array}} -->
The `options_is_array` function checks if a given options entry is of type array.
- **Inputs**:
    - `o`: A pointer to a `struct options_entry` which represents the options entry to be checked.
- **Control Flow**:
    - The function directly returns the result of the macro `OPTIONS_IS_ARRAY(o)`.
    - The macro checks if the `tableentry` of the options entry is not NULL and if the `flags` indicate that it is an array.
- **Output**:
    - Returns a non-zero integer if the options entry is an array, otherwise returns zero.


---
### options_is_string<!-- {{#callable:options_is_string}} -->
The `options_is_string` function checks if a given options entry is of type string.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure that represents the options entry to be checked.
- **Control Flow**:
    - The function calls the macro `OPTIONS_IS_STRING` with the input argument `o`.
    - The macro evaluates whether the `tableentry` of the options entry is NULL or if its type is `OPTIONS_TABLE_STRING`.
- **Output**:
    - Returns an integer value: 1 if the options entry is a string type, otherwise returns 0.


---
### options_to_string<!-- {{#callable:options_to_string}} -->
Converts an `options_entry` structure to a string representation.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure that contains the options to be converted.
    - `idx`: An integer index to specify which item in an array to convert; -1 indicates all items.
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
    - Searches for the corresponding ']' after the '[' to validate the index.
    - If ']' is not found, or if the character after ']' is not null, or if the character before ']' is not a digit, frees `copy` and returns NULL.
    - Uses `sscanf` to extract the integer index from the string and checks if it is non-negative.
    - If the index is valid, modifies `copy` to terminate the string at the '[' character and returns the modified string.
- **Output**:
    - Returns a pointer to the duplicated string without the index part if an index was found, or NULL if the input was invalid.


---
### options_parse_get<!-- {{#callable:options_parse_get}} -->
Parses an option string and retrieves the corresponding `options_entry` from the options structure.
- **Inputs**:
    - `oo`: A pointer to the `options` structure from which the option is to be retrieved.
    - `s`: A string representing the option name, which may include an index in square brackets.
    - `idx`: A pointer to an integer that will store the parsed index if present in the option string.
    - `only`: An integer flag indicating whether to retrieve only the exact match of the option.
- **Control Flow**:
    - Calls [`options_parse`](#options_parse) to extract the option name and index from the input string `s`.
    - If [`options_parse`](#options_parse) returns NULL, the function returns NULL, indicating that the parsing failed.
    - If the `only` flag is set, it calls [`options_get_only`](#options_get_only) to retrieve the option entry matching the name.
    - If the `only` flag is not set, it calls [`options_get`](#options_get) to search for the option entry in the current options and its parent options.
    - Frees the memory allocated for the name string before returning the found option entry.
- **Output**:
    - Returns a pointer to the `options_entry` corresponding to the parsed option name, or NULL if the option could not be found or parsed.
- **Functions called**:
    - [`options_parse`](#options_parse)
    - [`options_get_only`](#options_get_only)
    - [`options_get`](#options_get)


---
### options_match<!-- {{#callable:options_match}} -->
The `options_match` function attempts to match a given string against a predefined set of options and returns the matched option name.
- **Inputs**:
    - `s`: A constant character pointer representing the string to match against the options.
    - `idx`: A pointer to an integer that will hold the index parsed from the input string.
    - `ambiguous`: A pointer to an integer that indicates whether the match is ambiguous.
- **Control Flow**:
    - The function first calls [`options_parse`](#options_parse) to parse the input string `s` and retrieve a parsed string.
    - If the parsed string is NULL, the function returns NULL.
    - If the parsed string starts with '@', it sets `*ambiguous` to 0 and returns the parsed string.
    - The function then maps the parsed string to an option name using [`options_map_name`](#options_map_name).
    - It iterates through the `options_table` to find a matching option name.
    - If a match is found, it checks for ambiguity by looking for other options that match the beginning of the name.
    - If multiple matches are found, it sets `*ambiguous` to 1 and returns NULL.
    - If no matches are found, it sets `*ambiguous` to 0 and returns NULL.
    - If a single match is found, it duplicates the matched option name and returns it.
- **Output**:
    - The function returns a dynamically allocated string containing the matched option name, or NULL if no match is found or if the match is ambiguous.
- **Functions called**:
    - [`options_parse`](#options_parse)
    - [`options_map_name`](#options_map_name)


---
### options_match_get<!-- {{#callable:options_match_get}} -->
Retrieves an `options_entry` that matches a given string from a specified options structure.
- **Inputs**:
    - `oo`: A pointer to an `options` structure from which to retrieve the matching option.
    - `s`: A string representing the name of the option to match.
    - `idx`: A pointer to an integer that will store the index of the matched option if applicable.
    - `only`: An integer flag indicating whether to retrieve only the exact match.
    - `ambiguous`: A pointer to an integer that will be set to indicate if the match is ambiguous.
- **Control Flow**:
    - Calls the [`options_match`](#options_match) function to find a matching option name based on the input string `s`.
    - If no match is found (i.e., `name` is NULL), the function returns NULL.
    - Sets the `ambiguous` flag to 0 to indicate that the match is not ambiguous.
    - If the `only` flag is set, it calls [`options_get_only`](#options_get_only) to retrieve the exact match; otherwise, it calls [`options_get`](#options_get) to retrieve the match from the options structure.
    - Frees the allocated memory for the matched name before returning the result.
- **Output**:
    - Returns a pointer to the `options_entry` that matches the specified string, or NULL if no match is found.
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
    - `oo`: A pointer to the `options` structure from which the option is to be retrieved.
    - `name`: A string representing the name of the option to retrieve.
- **Control Flow**:
    - The function calls [`options_get`](#options_get) to find the `options_entry` associated with the provided `name` in the `options` structure.
    - If the `options_entry` is not found (i.e., `o` is NULL), it triggers a fatal error indicating that the option is missing.
    - If the found `options_entry` does not represent a number (checked using `OPTIONS_IS_NUMBER`), it triggers a fatal error indicating that the option is not a number.
    - If both checks pass, the function returns the numeric value stored in the `options_entry`.
- **Output**:
    - Returns the numeric value of the option as a `long long` type.
- **Functions called**:
    - [`options_get`](#options_get)


---
### options_get_command<!-- {{#callable:options_get_command}} -->
Retrieves a command option from the options structure.
- **Inputs**:
    - `oo`: A pointer to the `options` structure from which the command option is to be retrieved.
    - `name`: A string representing the name of the command option to retrieve.
- **Control Flow**:
    - Calls [`options_get`](#options_get) to retrieve the `options_entry` associated with the given `name` from the `options` structure.
    - If the retrieved `options_entry` is NULL, it triggers a fatal error indicating that the option is missing.
    - Checks if the retrieved option is a command using the `OPTIONS_IS_COMMAND` macro; if not, it triggers a fatal error indicating the option is not a command.
    - Returns the command list associated with the `options_entry`.
- **Output**:
    - Returns a pointer to the `cmd_list` associated with the specified command option.
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
    - Initializes a variable argument list and formats the new string value using `xvasprintf`.
    - Retrieves the existing option entry using [`options_get_only`](#options_get_only).
    - If the option exists and `append` is true, concatenates the existing string with the new value, using a separator if defined.
    - If the option does not exist and the name starts with '@', it adds a new option entry.
    - If the option does not exist, it retrieves the default option entry from the parent options.
    - Checks if the option is of string type; if not, it raises a fatal error.
    - Frees the old string value and assigns the new value to the option entry.
- **Output**:
    - Returns a pointer to the `options_entry` structure representing the updated option.
- **Functions called**:
    - [`options_get_only`](#options_get_only)
    - [`options_add`](#options_add)
    - [`options_default`](#options_default)
    - [`options_parent_table_entry`](#options_parent_table_entry)


---
### options_set_number<!-- {{#callable:options_set_number}} -->
Sets a numeric value for a specified option.
- **Inputs**:
    - `oo`: A pointer to the `options` structure that contains the options.
    - `name`: A string representing the name of the option to set.
    - `value`: A long long integer representing the numeric value to assign to the option.
- **Control Flow**:
    - Checks if the first character of `name` is '@', and if so, calls `fatalx` to terminate with an error message.
    - Calls [`options_get_only`](#options_get_only) to retrieve the `options_entry` associated with `name` from the `options` structure.
    - If the entry is not found, it attempts to get a default entry using [`options_default`](#options_default) and [`options_parent_table_entry`](#options_parent_table_entry).
    - If the entry is still not found, it returns NULL.
    - Checks if the retrieved entry is of type number using `OPTIONS_IS_NUMBER`, and if not, calls `fatalx` to terminate with an error message.
    - Sets the `number` field of the `value` union in the `options_entry` to the provided `value`.
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
    - `value`: A pointer to a `cmd_list` structure representing the command value to be assigned.
- **Control Flow**:
    - Checks if the `name` starts with '@', and if so, calls `fatalx` to terminate with an error message.
    - Retrieves the options entry associated with `name` using [`options_get_only`](#options_get_only).
    - If the entry is not found, it attempts to get the default entry from the parent options using [`options_default`](#options_default).
    - Checks if the retrieved options entry is a command using `OPTIONS_IS_COMMAND`, and if not, calls `fatalx` to terminate with an error message.
    - If the command list in the options entry is not NULL, it frees the existing command list using `cmd_list_free`.
    - Assigns the new command list `value` to the options entry and returns the updated entry.
- **Output**:
    - Returns a pointer to the updated `options_entry` structure, or NULL if the entry could not be created.
- **Functions called**:
    - [`options_get_only`](#options_get_only)
    - [`options_default`](#options_default)
    - [`options_parent_table_entry`](#options_parent_table_entry)


---
### options_scope_from_name<!-- {{#callable:options_scope_from_name}} -->
The `options_scope_from_name` function determines the scope of options based on a given name and updates the provided options pointer.
- **Inputs**:
    - `args`: A pointer to a struct `args` that contains command-line arguments.
    - `window`: An integer representing the window identifier.
    - `name`: A string representing the name of the option whose scope is to be determined.
    - `fs`: A pointer to a struct `cmd_find_state` that holds the current state of command finding.
    - `oo`: A pointer to a pointer of struct `options` where the found options will be stored.
    - `cause`: A pointer to a string that will hold an error message if an error occurs.
- **Control Flow**:
    - Checks if the name starts with '@', in which case it calls [`options_scope_from_flags`](#options_scope_from_flags) to handle special flags.
    - Iterates through the `options_table` to find the corresponding option entry for the provided name.
    - If the name is not found, it sets an error message in `cause` and returns `OPTIONS_TABLE_NONE`.
    - Based on the scope of the found option entry, it updates the `oo` pointer to point to the appropriate options structure.
    - Handles different scopes such as server, session, window, and pane, checking for the existence of each context and setting error messages as needed.
- **Output**:
    - Returns an integer representing the scope of the option, which can be one of the predefined scope constants (e.g., `OPTIONS_TABLE_SERVER`, `OPTIONS_TABLE_SESSION`, etc.). If an error occurs, it returns `OPTIONS_TABLE_NONE`.
- **Functions called**:
    - [`options_scope_from_flags`](#options_scope_from_flags)


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
    - If the 'p' flag is present, it checks if the current pane (`wp`) is NULL; if it is, it sets an error message in `cause` and returns `OPTIONS_TABLE_NONE`, otherwise sets `*oo` to the pane's options and returns `OPTIONS_TABLE_PANE`.
    - If the `window` parameter is true or the 'w' flag is present, it checks for the 'g' flag; if present, it sets `*oo` to `global_w_options` and returns `OPTIONS_TABLE_WINDOW`.
    - If the current window (`wl`) is NULL, it sets an error message in `cause` and returns `OPTIONS_TABLE_NONE`, otherwise sets `*oo` to the window's options and returns `OPTIONS_TABLE_WINDOW`.
    - If none of the above conditions are met, it checks for the 'g' flag; if present, it sets `*oo` to `global_s_options` and returns `OPTIONS_TABLE_SESSION`.
    - If the current session (`s`) is NULL, it sets an error message in `cause` and returns `OPTIONS_TABLE_NONE`, otherwise sets `*oo` to the session's options and returns `OPTIONS_TABLE_SESSION`.
- **Output**:
    - Returns an integer representing the options scope: `OPTIONS_TABLE_SERVER`, `OPTIONS_TABLE_PANE`, `OPTIONS_TABLE_WINDOW`, `OPTIONS_TABLE_SESSION`, or `OPTIONS_TABLE_NONE` if an error occurs.


---
### options_string_to_style<!-- {{#callable:options_string_to_style}} -->
Converts a string option to a `style` structure.
- **Inputs**:
    - `oo`: A pointer to the `options` structure containing the options.
    - `name`: A string representing the name of the option to retrieve.
    - `ft`: A pointer to a `format_tree` structure used for expanding the option string.
- **Control Flow**:
    - Retrieve the option entry corresponding to `name` from the `options` structure.
    - Check if the option entry is valid and of string type; if not, return NULL.
    - If the style is cached, return the cached style.
    - Log the current value of the option.
    - Set the default style based on `grid_default_cell`.
    - Determine if the option string contains format placeholders; if not, cache the style.
    - If `ft` is not NULL and the style is not cached, expand the option string using `format_expand`.
    - Parse the expanded string into the style structure; if parsing fails, free the expanded string and return NULL.
    - If `ft` is NULL or the style is cached, parse the original string directly into the style structure.
    - Return a pointer to the populated style structure.
- **Output**:
    - Returns a pointer to the `style` structure populated based on the option string, or NULL if an error occurs.
- **Functions called**:
    - [`options_get`](#options_get)


---
### options_from_string_check<!-- {{#callable:options_from_string_check}} -->
Checks the validity of an option value against predefined criteria.
- **Inputs**:
    - `oe`: A pointer to an `options_table_entry` structure that defines the option being checked.
    - `value`: A string representing the value to be validated.
    - `cause`: A pointer to a string that will hold an error message if validation fails.
- **Control Flow**:
    - If `oe` is NULL, the function returns 0, indicating no error.
    - If the option name is 'default-shell' and the value does not pass the `checkshell` function, an error message is generated and -1 is returned.
    - If the `oe->pattern` is not NULL and the value does not match the pattern using `fnmatch`, an error message is generated and -1 is returned.
    - If the option is of style type and the value does not contain a style indicator ('#{') and fails to parse as a style, an error message is generated and -1 is returned.
    - If all checks pass, the function returns 0, indicating success.
- **Output**:
    - Returns 0 if the value is valid, or -1 if there is an error, with an error message stored in `cause`.


---
### options_from_string_flag<!-- {{#callable:options_from_string_flag}} -->
The `options_from_string_flag` function sets a flag option based on a string input.
- **Inputs**:
    - `oo`: A pointer to the `options` structure that holds the options.
    - `name`: A string representing the name of the option to be set.
    - `value`: A string representing the value to be interpreted as a flag.
    - `cause`: A pointer to a string that will hold an error message if the function fails.
- **Control Flow**:
    - Check if the `value` is NULL or an empty string; if so, toggle the current flag state.
    - Compare the `value` against '1', 'on', or 'yes' to set the flag to true (1).
    - Compare the `value` against '0', 'off', or 'no' to set the flag to false (0).
    - If the `value` does not match any expected strings, set an error message in `cause` and return -1.
    - If the flag is successfully determined, call [`options_set_number`](#options_set_number) to update the option's value.
- **Output**:
    - Returns 0 on success, or -1 if an error occurs, with an error message set in `cause`.
- **Functions called**:
    - [`options_get_number`](#options_get_number)
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
    - If no match is found after the loop, an error message is formatted and stored in `cause`, and -1 is returned.
    - If a match is found, the index of the choice is returned.
- **Output**:
    - Returns the index of the matching choice if found, or -1 if the value is not found, with an error message in `cause`.


---
### options_from_string_choice<!-- {{#callable:options_from_string_choice}} -->
The `options_from_string_choice` function sets an option's value based on a string representation of a choice.
- **Inputs**:
    - `oe`: A pointer to the `options_table_entry` structure that defines the option's properties.
    - `oo`: A pointer to the `options` structure where the option value will be set.
    - `name`: A string representing the name of the option to be set.
    - `value`: A string representing the choice value to be set, or NULL to retrieve the current choice.
    - `cause`: A pointer to a string that will hold an error message if an error occurs.
- **Control Flow**:
    - If `value` is NULL, the function retrieves the current choice using [`options_get_number`](#options_get_number).
    - If the retrieved choice is less than 2, it toggles the choice to the opposite value.
    - If `value` is not NULL, it attempts to find the corresponding choice index using [`options_find_choice`](#options_find_choice).
    - If the choice index is valid (non-negative), it sets the option's value using [`options_set_number`](#options_set_number).
    - If the choice index is invalid (negative), it returns -1 to indicate an error.
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
    - The function first checks if the `oe` pointer is not NULL to determine the option type.
    - If `value` is NULL and the option type is not a flag or choice, an error message is generated.
    - The function then uses a switch statement to handle different option types: string, number, key, colour, flag, choice, and command.
    - For string options, it duplicates the old value, sets the new value, and checks for validity before committing the change.
    - For number options, it converts the string to a number and checks if it falls within the specified range.
    - For key options, it looks up the key code from the string value.
    - For colour options, it converts the string to a colour value.
    - For flag options, it toggles the flag based on the string value.
    - For choice options, it finds the corresponding choice index.
    - For command options, it parses the command string and sets it in the options structure.
- **Output**:
    - The function returns 0 on success, or -1 if an error occurs, with an error message stored in `cause`.
- **Functions called**:
    - [`options_get_string`](#options_get_string)
    - [`options_set_string`](#options_set_string)
    - [`options_from_string_check`](#options_from_string_check)
    - [`options_set_number`](#options_set_number)
    - [`options_from_string_flag`](#options_from_string_flag)
    - [`options_from_string_choice`](#options_from_string_choice)
    - [`options_set_command`](#options_set_command)


---
### options_push_changes<!-- {{#callable:options_push_changes}} -->
The `options_push_changes` function updates various settings based on the provided option name.
- **Inputs**:
    - `name`: A string representing the name of the option whose changes need to be pushed.
- **Control Flow**:
    - Logs the function call with the provided option name.
    - Checks if the option name matches specific cases (e.g., 'automatic-rename', 'cursor-colour', etc.) and executes corresponding actions.
    - For 'automatic-rename', iterates through all windows and marks the active pane as changed if the option is enabled.
    - For 'cursor-colour' and 'cursor-style', updates the default cursor for all window panes.
    - For 'fill-character', sets the fill character for all windows.
    - For 'key-table', updates the key table for all clients.
    - For 'user-keys', rebuilds the key bindings for clients with an open terminal.
    - For 'status' and 'status-interval', starts the status timer for all sessions.
    - For 'monitor-silence', resets all alerts.
    - For 'window-style' and 'window-active-style', marks the style and theme as changed for all window panes.
    - For 'pane-colours', updates the color palette for all window panes.
    - For 'pane-border-status', 'pane-scrollbars', and 'pane-scrollbars-position', fixes the layout of panes in all windows.
    - For 'pane-scrollbars-style', sets the scrollbar style for all window panes and fixes the layout of panes.
    - For 'codepoint-widths', updates the UTF-8 width cache.
    - For 'input-buffer-size', sets the input buffer size based on the global options.
    - Updates the status cache for all sessions.
    - Recalculates sizes and redraws clients with active sessions.
- **Output**:
    - The function does not return a value but modifies the state of various components based on the option name.
- **Functions called**:
    - [`options_get_number`](#options_get_number)


---
### options_remove_or_default<!-- {{#callable:options_remove_or_default}} -->
Removes an option or sets it to its default value based on the provided index.
- **Inputs**:
    - `o`: A pointer to the `options_entry` structure representing the option to be modified.
    - `idx`: An integer index indicating the specific array element to modify; if -1, the entire option is affected.
    - `cause`: A pointer to a character pointer that can be used to return an error message if the operation fails.
- **Control Flow**:
    - Checks if the index `idx` is -1, indicating that the entire option should be removed or reset to default.
    - If `idx` is -1 and the option has a `tableentry`, it checks if the owner of the option is one of the global options and calls [`options_default`](#options_default) to reset it.
    - If the owner is not a global option, it calls [`options_remove`](#options_remove) to remove the option.
    - If `idx` is not -1, it attempts to set the array element at the specified index to NULL using [`options_array_set`](#options_array_set), returning -1 if it fails.
- **Output**:
    - Returns 0 on success, or -1 if an error occurs during the operation.
- **Functions called**:
    - [`options_default`](#options_default)
    - [`options_remove`](#options_remove)
    - [`options_array_set`](#options_array_set)


# Function Declarations (Public API)

---
### options_add<!-- {{#callable_declaration:options_add}} -->
Adds or replaces an option entry in the options tree.
- **Description**: This function is used to add a new option entry to the specified options tree or replace an existing entry with the same name. It is typically called when a new option needs to be defined or an existing option needs to be updated. The function ensures that any existing entry with the same name is removed before adding the new entry. It is important to ensure that the options structure is properly initialized before calling this function.
- **Inputs**:
    - `oo`: A pointer to the options structure where the entry will be added. Must not be null and should be properly initialized.
    - `name`: A string representing the name of the option to be added. Must not be null. The function will duplicate this string for internal use.
- **Output**: Returns a pointer to the newly added or replaced options_entry structure.
- **See also**: [`options_add`](#options_add)  (Implementation)


---
### options_remove<!-- {{#callable_declaration:options_remove}} -->
Removes an option entry from its owner.
- **Description**: Use this function to remove an option entry from its owner, effectively deleting it from the options tree. This function should be called when an option is no longer needed or should be cleared. It handles both array and non-array options, ensuring that any associated resources are properly freed. The function must be called with a valid `options_entry` pointer, and it assumes ownership of the memory associated with the entry, freeing it as part of the removal process.
- **Inputs**:
    - `o`: A pointer to the `options_entry` to be removed. Must not be null and should point to a valid entry that is part of an options tree. The function will free the memory associated with this entry.
- **Output**: None
- **See also**: [`options_remove`](#options_remove)  (Implementation)


---
### options_cmp<!-- {{#callable_declaration:options_cmp}} -->
Compares two options entries by their names.
- **Description**: Use this function to compare two options entries based on their names, which are expected to be strings. This function is useful when sorting or searching through a collection of options entries. It assumes that both input parameters are valid pointers to options_entry structures and that their name fields are non-null strings. The function does not handle null pointers or invalid data gracefully, so ensure that the inputs are properly initialized before calling this function.
- **Inputs**:
    - `lhs`: A pointer to the first options_entry structure to compare. Must not be null and must have a valid name field.
    - `rhs`: A pointer to the second options_entry structure to compare. Must not be null and must have a valid name field.
- **Output**: Returns an integer less than, equal to, or greater than zero if the name of the first options_entry is found, respectively, to be less than, to match, or be greater than the name of the second options_entry.
- **See also**: [`options_cmp`](#options_cmp)  (Implementation)


