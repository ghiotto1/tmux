# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and is responsible for handling command-line arguments. The file defines structures and functions to parse, store, manipulate, and retrieve command-line arguments and their associated values. It provides a comprehensive set of utilities for managing arguments, including parsing flags and their optional or required values, converting arguments to and from string representations, and handling argument types such as strings and command lists. The code also includes functionality for copying, expanding, and freeing argument sets, as well as converting arguments to numbers, including handling percentages and expanding formats.

The file is structured around several key data structures, such as `args`, `args_entry`, and [`args_value`](#args_value), which represent parsed arguments, individual argument flags, and their values, respectively. It uses a red-black tree (`args_tree`) to efficiently manage and search for argument entries. The code provides a public API for creating, parsing, and manipulating argument sets, making it a crucial component for command processing within tmux. The functions are designed to be used internally by other parts of the tmux codebase, facilitating the handling of complex command-line input and ensuring robust argument management.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### args_find
- **Type**: `static struct args_entry*`
- **Description**: The `args_find` variable is a static function pointer that returns a pointer to an `args_entry` structure. It takes two parameters: a pointer to an `args` structure and an unsigned character representing a flag.
- **Use**: This function is used to locate a specific flag within the arguments tree of an `args` structure.


# Data Structures

---
### args_entry
- **Type**: `struct`
- **Members**:
    - `flag`: A single character flag represented as an unsigned char.
    - `values`: A list of argument values, defined as a TAILQ_HEAD of args_value.
    - `count`: An unsigned integer representing the number of values associated with the flag.
    - `flags`: An integer to store additional flags, such as ARGS_ENTRY_OPTIONAL_VALUE.
    - `entry`: A Red-Black tree entry for organizing args_entry structures.
- **Description**: The `args_entry` structure is used to represent a single command-line argument flag and its associated values in a command-line parsing context. It includes a flag character, a list of values associated with the flag, a count of these values, and additional flags to indicate optional values. The structure is organized within a Red-Black tree to facilitate efficient searching and management of multiple argument entries.


---
### args
- **Type**: `struct`
- **Members**:
    - `tree`: A tree structure to store parsed argument flags and values.
    - `count`: The number of argument values stored.
    - `values`: A pointer to an array of argument values.
- **Description**: The `args` structure is designed to manage command-line arguments in a structured manner. It contains a tree (`args_tree`) to efficiently store and retrieve argument flags and their associated values, a count of the total number of argument values, and a pointer to an array of `args_value` structures that hold the actual argument data. This structure is part of a larger system for parsing and handling command-line arguments, allowing for complex argument parsing and manipulation.


---
### args_command_state
- **Type**: `struct`
- **Members**:
    - `cmdlist`: A pointer to a list of commands.
    - `cmd`: A pointer to a character string representing a command.
    - `pi`: An instance of the cmd_parse_input structure, used for parsing command input.
- **Description**: The `args_command_state` structure is used to maintain the state of a command being processed. It includes a command list (`cmdlist`) which holds the commands to be executed, a command string (`cmd`) that represents the command itself, and a `cmd_parse_input` structure (`pi`) that is used to handle the parsing of command input. This structure is essential for managing and executing commands within the context of the application, providing a way to store and manipulate command-related data.


# Functions

---
### args_cmp<!-- {{#callable:args_cmp}} -->
The `args_cmp` function compares two `args_entry` structures based on their `flag` values.
- **Inputs**:
    - `a1`: A pointer to the first `args_entry` structure to be compared.
    - `a2`: A pointer to the second `args_entry` structure to be compared.
- **Control Flow**:
    - The function calculates the difference between the `flag` values of the two `args_entry` structures `a1` and `a2`.
    - It returns the result of the subtraction `a1->flag - a2->flag`.
- **Output**:
    - The function returns an integer that indicates the relative ordering of the two `args_entry` structures based on their `flag` values; a negative value if `a1`'s flag is less than `a2`'s, zero if they are equal, and a positive value if `a1`'s flag is greater than `a2`'s.


---
### args_find<!-- {{#callable:args_find}} -->
The `args_find` function searches for an argument entry with a specific flag in an argument tree.
- **Inputs**:
    - `args`: A pointer to a `struct args` which contains the argument tree to search.
    - `flag`: An `u_char` representing the flag of the argument entry to find.
- **Control Flow**:
    - A temporary `args_entry` structure is created and its `flag` field is set to the provided `flag` argument.
    - The `RB_FIND` function is called with the `args_tree`, the tree from the `args` structure, and the temporary `args_entry` to locate the entry with the matching flag.
- **Output**:
    - Returns a pointer to the `args_entry` structure if found, or `NULL` if no matching entry is found.


---
### args_copy_value<!-- {{#callable:args_copy_value}} -->
The `args_copy_value` function copies the value from one `args_value` structure to another, handling different types of values appropriately.
- **Inputs**:
    - `to`: A pointer to the `args_value` structure where the value will be copied to.
    - `from`: A pointer to the `args_value` structure from which the value will be copied.
- **Control Flow**:
    - The function starts by copying the `type` field from the `from` structure to the `to` structure.
    - A switch statement is used to handle different types of values based on the `type` field.
    - If the type is `ARGS_NONE`, no action is taken.
    - If the type is `ARGS_COMMANDS`, the `cmdlist` pointer is copied from `from` to `to`, and the `references` count of the `cmdlist` is incremented.
    - If the type is `ARGS_STRING`, the `string` field is duplicated using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and assigned to the `to` structure.
- **Output**:
    - The function does not return a value; it modifies the `to` structure in place.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### args_type_to_string<!-- {{#callable:args_type_to_string}} -->
The `args_type_to_string` function converts an `args_type` enumeration value to its corresponding string representation.
- **Inputs**:
    - `type`: An enumeration value of type `args_type` which can be one of `ARGS_NONE`, `ARGS_STRING`, or `ARGS_COMMANDS`.
- **Control Flow**:
    - The function uses a switch statement to determine the string representation of the input `type`.
    - If `type` is `ARGS_NONE`, it returns the string "NONE".
    - If `type` is `ARGS_STRING`, it returns the string "STRING".
    - If `type` is `ARGS_COMMANDS`, it returns the string "COMMANDS".
    - If `type` does not match any of the known cases, it returns the string "INVALID".
- **Output**:
    - A constant character pointer to the string representation of the input `args_type` value.


---
### args_value_as_string<!-- {{#callable:args_value_as_string}} -->
The `args_value_as_string` function converts an `args_value` structure to a string representation based on its type.
- **Inputs**:
    - `value`: A pointer to a `struct args_value` which contains the type and data to be converted to a string.
- **Control Flow**:
    - The function uses a switch statement to determine the type of the `args_value` structure.
    - If the type is `ARGS_NONE`, it returns an empty string.
    - If the type is `ARGS_COMMANDS`, it checks if the `cached` field is `NULL`; if so, it populates it using [`cmd_list_print`](cmd.c.driver.md#cmd_list_print) and then returns the `cached` value.
    - If the type is `ARGS_STRING`, it directly returns the `string` field of the `args_value`.
    - If the type is not recognized, it calls `fatalx` with an error message indicating an unexpected argument type.
- **Output**:
    - The function returns a constant character pointer to the string representation of the `args_value`.
- **Functions called**:
    - [`cmd_list_print`](cmd.c.driver.md#cmd_list_print)


---
### args_create<!-- {{#callable:args_create}} -->
The `args_create` function initializes and returns a new, empty `args` structure with a red-black tree for managing argument entries.
- **Inputs**:
    - None
- **Control Flow**:
    - Allocate memory for a new `args` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc), initializing it to zero.
    - Initialize the red-black tree within the `args` structure using `RB_INIT`.
    - Return the newly created `args` structure.
- **Output**:
    - A pointer to a newly allocated and initialized `args` structure.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### args_parse_flag_argument<!-- {{#callable:args_parse_flag_argument}} -->
The `args_parse_flag_argument` function processes a single flag argument, handling both required and optional string arguments, and updates the argument set accordingly.
- **Inputs**:
    - `values`: An array of `args_value` structures representing the parsed argument values.
    - `count`: The total number of arguments in the `values` array.
    - `cause`: A pointer to a string that will store an error message if the function encounters an error.
    - `args`: A pointer to an `args` structure where the parsed flag and its value will be stored.
    - `i`: A pointer to an unsigned integer representing the current index in the `values` array.
    - `string`: A constant character pointer representing the string value of the flag argument.
    - `flag`: An integer representing the flag character being processed.
    - `optional_argument`: An integer indicating whether the flag argument is optional (non-zero) or required (zero).
- **Control Flow**:
    - Allocate memory for a new `args_value` structure and initialize it.
    - Check if the `string` is not empty; if so, set the new value's type to `ARGS_STRING` and duplicate the string, then proceed to the output section.
    - If the current index `i` equals `count`, set `argument` to NULL; otherwise, set `argument` to the current value in the `values` array.
    - If `argument` is not a string, set an error message in `cause`, free the new value, and return -1.
    - If `argument` is NULL and the argument is optional, log a debug message, set the flag in `args` with an optional value, and return 0.
    - If `argument` is NULL and the argument is not optional, set an error message in `cause` and return -1.
    - Copy the `argument` value to the new value, increment the index `i`, and proceed to the output section.
    - Convert the new value to a string, log a debug message, set the flag in `args` with the new value, and return 0.
- **Output**:
    - Returns 0 on success, or -1 if an error occurs, with an error message set in `cause`.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`args_free_value`](#args_free_value)
    - [`args_set`](#args_set)
    - [`args_copy_value`](#args_copy_value)
    - [`args_value_as_string`](#args_value_as_string)


---
### args_parse_flags<!-- {{#callable:args_parse_flags}} -->
The `args_parse_flags` function parses command-line flags from a list of argument values and updates the provided `args` structure accordingly.
- **Inputs**:
    - `parse`: A pointer to an `args_parse` structure that contains the template for valid flags.
    - `values`: An array of `args_value` structures representing the argument values to be parsed.
    - `count`: The total number of argument values in the `values` array.
    - `cause`: A pointer to a string that will be set with an error message if parsing fails.
    - `args`: A pointer to an `args` structure where parsed flags and their values will be stored.
    - `i`: A pointer to an unsigned integer that tracks the current index in the `values` array.
- **Control Flow**:
    - Retrieve the current argument value from the `values` array using the index `*i` and check if its type is `ARGS_STRING`; if not, return 1 indicating no flags to parse.
    - Check if the string starts with a '-' character; if not, return 1 indicating no flags to parse.
    - Increment the index `*i` to move to the next argument value.
    - Enter a loop to process each character in the string as a potential flag.
    - For each character, check if it is a valid alphanumeric flag; if not, set an error message in `cause` and return -1.
    - Check if the flag is present in the `parse->template`; if not, set an error message in `cause` and return -1.
    - If the flag requires an argument (indicated by a ':' in the template), call [`args_parse_flag_argument`](#args_parse_flag_argument) to handle the argument parsing.
    - If the flag does not require an argument, add it to the `args` structure with a NULL value.
- **Output**:
    - Returns 0 on successful parsing of flags, 1 if no flags are found, and -1 if an error occurs during parsing.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`args_set`](#args_set)
    - [`args_parse_flag_argument`](#args_parse_flag_argument)


---
### args_parse<!-- {{#callable:args_parse}} -->
The `args_parse` function parses command-line arguments based on a given template and returns a structured set of parsed arguments.
- **Inputs**:
    - `parse`: A pointer to a `struct args_parse` which contains the parsing template and callback function.
    - `values`: An array of `struct args_value` representing the command-line arguments to be parsed.
    - `count`: The number of elements in the `values` array.
    - `cause`: A pointer to a string that will store an error message if parsing fails.
- **Control Flow**:
    - If `count` is zero, create and return an empty `args` structure.
    - Initialize a new `args` structure to store parsed arguments.
    - Iterate over the `values` array starting from index 1, parsing flags using [`args_parse_flags`](#args_parse_flags).
    - If [`args_parse_flags`](#args_parse_flags) returns -1, free the `args` structure and return NULL due to an error.
    - If [`args_parse_flags`](#args_parse_flags) returns 1, break the loop as flag parsing is complete.
    - Log the end of flag parsing and continue parsing remaining arguments if any.
    - For each remaining argument, convert it to a string and log it.
    - If a callback is provided in `parse`, use it to determine the argument type; otherwise, default to `ARGS_PARSE_STRING`.
    - Reallocate the `args->values` array to accommodate a new argument value.
    - Depending on the determined type, validate and copy the argument value, handling errors by freeing `args` and returning NULL.
    - After parsing all arguments, check if the number of parsed arguments is within the specified bounds (`lower` and `upper` in `parse`).
    - If the number of arguments is out of bounds, set an error message in `cause`, free `args`, and return NULL.
    - Return the populated `args` structure if parsing is successful.
- **Output**:
    - Returns a pointer to a `struct args` containing the parsed arguments, or NULL if an error occurs during parsing.
- **Functions called**:
    - [`args_create`](#args_create)
    - [`args_parse_flags`](#args_parse_flags)
    - [`args_free`](#args_free)
    - [`args_value_as_string`](#args_value_as_string)
    - [`args_type_to_string`](#args_type_to_string)
    - [`xrecallocarray`](xmalloc.c.driver.md#xrecallocarray)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`args_copy_value`](#args_copy_value)


---
### args_copy_copy_value<!-- {{#callable:args_copy_copy_value}} -->
The `args_copy_copy_value` function copies and potentially expands a value from one `args_value` structure to another, based on the type of the value and additional arguments provided.
- **Inputs**:
    - `to`: A pointer to the `args_value` structure where the value will be copied to.
    - `from`: A pointer to the `args_value` structure from which the value will be copied.
    - `argc`: An integer representing the number of arguments in the `argv` array.
    - `argv`: An array of strings used for template replacement in the value copying process.
- **Control Flow**:
    - The function begins by setting the `type` of the `to` structure to match the `type` of the `from` structure.
    - A switch statement is used to handle different types of values (`ARGS_NONE`, `ARGS_STRING`, `ARGS_COMMANDS`).
    - For `ARGS_NONE`, no action is taken.
    - For `ARGS_STRING`, the string from `from` is duplicated and stored in `expanded`.
    - A loop iterates over `argc`, replacing parts of `expanded` using [`cmd_template_replace`](cmd.c.driver.md#cmd_template_replace) with each `argv` element, updating `expanded` each time.
    - The final expanded string is assigned to `to->string`.
    - For `ARGS_COMMANDS`, the command list from `from` is copied to `to` using [`cmd_list_copy`](cmd.c.driver.md#cmd_list_copy), passing `argc` and `argv` for potential expansion.
- **Output**:
    - The function does not return a value; it modifies the `to` structure in place.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`cmd_template_replace`](cmd.c.driver.md#cmd_template_replace)
    - [`cmd_list_copy`](cmd.c.driver.md#cmd_list_copy)


---
### args_copy<!-- {{#callable:args_copy}} -->
The `args_copy` function creates a deep copy of an `args` structure, including its entries and values, while expanding any templates using the provided `argc` and `argv`.
- **Inputs**:
    - `args`: A pointer to the `args` structure to be copied.
    - `argc`: An integer representing the number of arguments in `argv`.
    - `argv`: An array of strings representing the arguments to be used for template expansion.
- **Control Flow**:
    - Log the function call with the provided arguments using `cmd_log_argv`.
    - Create a new `args` structure using [`args_create`](#args_create).
    - Iterate over each entry in the `args` tree using `RB_FOREACH`.
    - For each entry, check if it has any values using `TAILQ_EMPTY`.
    - If no values are present, replicate the entry's flag in the new `args` structure for the number of times specified by `entry->count`.
    - If values are present, iterate over them using `TAILQ_FOREACH`.
    - For each value, allocate memory for a new value and copy the original value into it using [`args_copy_copy_value`](#args_copy_copy_value), expanding templates with `argc` and `argv`.
    - Set the copied value in the new `args` structure using [`args_set`](#args_set).
    - If the original `args` structure has a non-zero count, allocate memory for the new `args` values and copy each value from the original `args` structure, expanding templates as needed.
    - Return the newly created `args` structure.
- **Output**:
    - A pointer to the newly created `args` structure, which is a deep copy of the input `args` structure with expanded templates.
- **Functions called**:
    - [`args_create`](#args_create)
    - [`args_set`](#args_set)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`args_copy_copy_value`](#args_copy_copy_value)


---
### args_free_value<!-- {{#callable:args_free_value}} -->
The `args_free_value` function deallocates memory associated with an `args_value` structure based on its type.
- **Inputs**:
    - `value`: A pointer to an `args_value` structure whose resources need to be freed.
- **Control Flow**:
    - The function checks the type of the `args_value` structure using a switch statement.
    - If the type is `ARGS_NONE`, no action is taken.
    - If the type is `ARGS_STRING`, the memory allocated for the `string` field is freed.
    - If the type is `ARGS_COMMANDS`, the [`cmd_list_free`](cmd.c.driver.md#cmd_list_free) function is called to free the `cmdlist` field.
    - Finally, the `cached` field of the `args_value` structure is freed regardless of the type.
- **Output**:
    - The function does not return any value; it performs memory deallocation for the specified `args_value` structure.
- **Functions called**:
    - [`cmd_list_free`](cmd.c.driver.md#cmd_list_free)


---
### args_free_values<!-- {{#callable:args_free_values}} -->
The `args_free_values` function iterates over an array of `args_value` structures and frees each one using the [`args_free_value`](#args_free_value) function.
- **Inputs**:
    - `values`: A pointer to an array of `args_value` structures that need to be freed.
    - `count`: The number of elements in the `values` array.
- **Control Flow**:
    - Initialize a loop counter `i` to 0.
    - Iterate over the `values` array from index 0 to `count - 1`.
    - For each element in the array, call [`args_free_value`](#args_free_value) to free the resources associated with that element.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided array of `args_value` structures.
- **Functions called**:
    - [`args_free_value`](#args_free_value)


---
### args_free<!-- {{#callable:args_free}} -->
The `args_free` function deallocates memory associated with an `args` structure, including its entries and values.
- **Inputs**:
    - `args`: A pointer to a `struct args` which contains a tree of argument entries and an array of argument values to be freed.
- **Control Flow**:
    - Call [`args_free_values`](#args_free_values) to free the array of argument values and then free the array itself.
    - Iterate over each entry in the `args` tree using `RB_FOREACH_SAFE`, removing each entry from the tree.
    - For each entry, iterate over its values using `TAILQ_FOREACH_SAFE`, removing each value from the entry's values list, freeing the value using [`args_free_value`](#args_free_value), and then freeing the value itself.
    - Free each entry after its values have been freed.
    - Finally, free the `args` structure itself.
- **Output**:
    - The function does not return any value; it performs memory deallocation for the `args` structure and its components.
- **Functions called**:
    - [`args_free_values`](#args_free_values)
    - [`args_free_value`](#args_free_value)


---
### args_to_vector<!-- {{#callable:args_to_vector}} -->
The `args_to_vector` function converts a structured list of arguments into a standard argument vector format, updating the argument count and vector accordingly.
- **Inputs**:
    - `args`: A pointer to a `struct args` which contains a list of argument values to be converted.
    - `argc`: A pointer to an integer where the function will store the count of arguments in the resulting vector.
    - `argv`: A pointer to a pointer to a char array where the function will store the resulting argument vector.
- **Control Flow**:
    - Initialize `*argc` to 0 and `*argv` to NULL to prepare for argument vector construction.
    - Iterate over each argument in the `args` structure using a for loop.
    - For each argument, check its type using a switch statement.
    - If the argument type is `ARGS_NONE`, do nothing and continue to the next argument.
    - If the argument type is `ARGS_STRING`, append the string to the argument vector using [`cmd_append_argv`](cmd.c.driver.md#cmd_append_argv).
    - If the argument type is `ARGS_COMMANDS`, convert the command list to a string using [`cmd_list_print`](cmd.c.driver.md#cmd_list_print), append it to the argument vector, and free the temporary string.
- **Output**:
    - The function updates `*argc` with the number of arguments and `*argv` with the constructed argument vector.
- **Functions called**:
    - [`cmd_append_argv`](cmd.c.driver.md#cmd_append_argv)
    - [`cmd_list_print`](cmd.c.driver.md#cmd_list_print)


---
### args_from_vector<!-- {{#callable:args_from_vector}} -->
The `args_from_vector` function converts an array of command-line arguments into an array of `args_value` structures, each representing a string argument.
- **Inputs**:
    - `argc`: The number of command-line arguments.
    - `argv`: An array of strings representing the command-line arguments.
- **Control Flow**:
    - Allocate memory for an array of `args_value` structures using [`xcalloc`](xmalloc.c.driver.md#xcalloc), with the size based on the number of arguments (`argc`).
    - Iterate over each argument in `argv` using a for loop.
    - For each argument, set the `type` field of the corresponding `args_value` structure to `ARGS_STRING`.
    - Duplicate the string from `argv` using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and assign it to the `string` field of the `args_value` structure.
    - Return the populated array of `args_value` structures.
- **Output**:
    - Returns a pointer to an array of `args_value` structures, each containing a duplicated string from the input `argv` array.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### printflike<!-- {{#callable:printflike}} -->
The `args_print_add` function appends a formatted string to a buffer, reallocating the buffer as necessary to accommodate the new content.
- **Inputs**:
    - `buf`: A pointer to a character buffer where the formatted string will be appended.
    - `len`: A pointer to a size_t variable that holds the current length of the buffer and will be updated to reflect the new length after appending.
    - `fmt`: A format string similar to those used in printf-style functions, specifying how the subsequent arguments should be formatted.
    - `...`: A variable number of arguments that will be formatted according to the format string `fmt`.
- **Control Flow**:
    - Initialize a variable argument list `ap` using `va_start` with the format string `fmt`.
    - Call [`xvasprintf`](xmalloc.c.driver.md#xvasprintf) to format the string according to `fmt` and store the result in `s`, updating `slen` with the length of the formatted string.
    - End the variable argument list using `va_end`.
    - Increase the length of the buffer `*len` by the length of the formatted string `slen`.
    - Reallocate the buffer `*buf` to the new length using [`xrealloc`](xmalloc.c.driver.md#xrealloc).
    - Concatenate the formatted string `s` to the buffer `*buf` using `strlcat`.
    - Free the memory allocated for the formatted string `s`.
- **Output**:
    - The function does not return a value, but it modifies the buffer `*buf` to include the newly formatted string and updates `*len` to reflect the new total length of the buffer.
- **Functions called**:
    - [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)
    - [`cmd_list_print`](cmd.c.driver.md#cmd_list_print)
    - [`args_escape`](#args_escape)


---
### args_print<!-- {{#callable:args_print}} -->
The `args_print` function generates a string representation of command-line arguments, including flags and their associated values, from a given `args` structure.
- **Inputs**:
    - `args`: A pointer to a `struct args` which contains the parsed command-line arguments, including flags and their values.
- **Control Flow**:
    - Initialize a buffer `buf` with a length of 1 to store the resulting string.
    - Iterate over the entries in the `args` tree to process flags without optional values, appending them to `buf`.
    - Iterate again over the entries to process flags with optional values or arguments, appending them and their values to `buf`.
    - If the last processed entry has an optional value flag, append a double dash `--` to `buf`.
    - Finally, append any remaining argument values from the `args` vector to `buf`.
- **Output**:
    - Returns a dynamically allocated string containing the formatted command-line arguments, which must be freed by the caller.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### args_escape<!-- {{#callable:args_escape}} -->
The `args_escape` function escapes a given string for safe inclusion in shell commands by adding necessary quotes or backslashes.
- **Inputs**:
    - `s`: A constant character pointer to the input string that needs to be escaped.
- **Control Flow**:
    - Check if the input string is empty; if so, return a pair of single quotes indicating an empty string.
    - Determine if the string contains characters that require double or single quotes by checking against predefined sets of characters.
    - If the string is a single character that is not a space and either requires quotes or is a tilde, escape it with a backslash and return.
    - Set flags for escaping special characters using octal, C-style, tab, and newline representations, and add double quote escaping if needed.
    - Use [`utf8_stravis`](utf8.c.driver.md#utf8_stravis) to escape the string based on the flags set.
    - If single quotes are needed, wrap the escaped string in single quotes; if double quotes are needed, wrap in double quotes, escaping a leading tilde if present.
    - If no quotes are needed, escape a leading tilde with a backslash or duplicate the escaped string.
    - Free the memory allocated for the escaped string and return the final result.
- **Output**:
    - A dynamically allocated string that is the escaped version of the input, suitable for safe inclusion in shell commands.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`utf8_stravis`](utf8.c.driver.md#utf8_stravis)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### args_has<!-- {{#callable:args_has}} -->
The `args_has` function checks if a specific flag is present in the given arguments structure and returns the count of its occurrences.
- **Inputs**:
    - `args`: A pointer to a `struct args` which represents the parsed argument flags and values.
    - `flag`: An unsigned character representing the flag to be checked within the arguments.
- **Control Flow**:
    - Call the [`args_find`](#args_find) function to locate the `args_entry` associated with the given flag in the `args` structure.
    - Check if the returned `args_entry` is `NULL`. If it is, return 0 indicating the flag is not present.
    - If the `args_entry` is not `NULL`, return the `count` field of the `args_entry`, which indicates the number of times the flag is present.
- **Output**:
    - Returns an integer representing the count of occurrences of the specified flag in the arguments, or 0 if the flag is not present.
- **Functions called**:
    - [`args_find`](#args_find)


---
### args_set<!-- {{#callable:args_set}} -->
The `args_set` function adds or updates an argument entry in an argument set, associating a flag with a value and managing its count and flags.
- **Inputs**:
    - `args`: A pointer to a `struct args` which represents the set of arguments where the flag and value will be stored.
    - `flag`: An `u_char` representing the flag to be set or updated in the argument set.
    - `value`: A pointer to a `struct args_value` representing the value associated with the flag, which can be NULL or have a type of `ARGS_NONE`.
    - `flags`: An `int` representing additional flags for the argument entry, such as `ARGS_ENTRY_OPTIONAL_VALUE`.
- **Control Flow**:
    - The function first attempts to find an existing entry for the given flag using [`args_find`](#args_find).
    - If no entry is found, a new `args_entry` is allocated and initialized with the flag, count set to 1, and the provided flags.
    - The new entry is inserted into the argument tree of the `args` structure using `RB_INSERT`.
    - If an entry is found, its count is incremented.
    - If the `value` is not NULL and its type is not `ARGS_NONE`, it is added to the entry's values list using `TAILQ_INSERT_TAIL`.
    - If the `value` is NULL or its type is `ARGS_NONE`, the `value` is freed.
- **Output**:
    - The function does not return a value; it modifies the `args` structure in place by adding or updating an entry for the specified flag.
- **Functions called**:
    - [`args_find`](#args_find)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### args_get<!-- {{#callable:args_get}} -->
The `args_get` function retrieves the last string value associated with a specified flag from a given set of arguments.
- **Inputs**:
    - `args`: A pointer to a `struct args` which represents a set of parsed argument flags and values.
    - `flag`: An unsigned character representing the specific flag whose associated value is to be retrieved.
- **Control Flow**:
    - The function first attempts to find an `args_entry` corresponding to the given flag using the [`args_find`](#args_find) function.
    - If no entry is found, the function returns `NULL`.
    - If the entry is found but contains no values, the function returns `NULL`.
    - If the entry contains values, the function returns the string of the last value in the entry's values list.
- **Output**:
    - Returns a `const char *` which is the string of the last value associated with the specified flag, or `NULL` if the flag is not found or has no values.
- **Functions called**:
    - [`args_find`](#args_find)


---
### args_first<!-- {{#callable:args_first}} -->
The `args_first` function retrieves the first entry from an argument tree and returns its flag.
- **Inputs**:
    - `args`: A pointer to a `struct args` which contains the argument tree to be searched.
    - `entry`: A pointer to a `struct args_entry*` where the first entry of the tree will be stored.
- **Control Flow**:
    - The function uses `RB_MIN` to find the minimum (first) entry in the `args_tree` within the `args` structure.
    - It assigns this entry to the location pointed to by `entry`.
    - If the entry is `NULL`, indicating the tree is empty, the function returns `0`.
    - If an entry is found, the function returns the `flag` of the first entry.
- **Output**:
    - The function returns a `u_char` representing the flag of the first entry in the argument tree, or `0` if the tree is empty.


---
### args_next<!-- {{#callable:args_next}} -->
The `args_next` function retrieves the next entry in an argument tree and returns its flag.
- **Inputs**:
    - `entry`: A pointer to a pointer of type `struct args_entry`, representing the current entry in the argument tree.
- **Control Flow**:
    - The function uses `RB_NEXT` to get the next entry in the `args_tree` from the current entry pointed to by `*entry`.
    - If the next entry is `NULL`, the function returns `0`.
    - If the next entry is not `NULL`, the function returns the `flag` of the next entry.
- **Output**:
    - The function returns a `u_char` representing the flag of the next entry in the argument tree, or `0` if there is no next entry.


---
### args_count<!-- {{#callable:args_count}} -->
The `args_count` function returns the number of arguments in a given `args` structure.
- **Inputs**:
    - `args`: A pointer to a `struct args` which contains the arguments and their count.
- **Control Flow**:
    - The function directly accesses the `count` member of the `args` structure and returns its value.
- **Output**:
    - The function returns an unsigned integer representing the number of arguments in the `args` structure.


---
### args_values<!-- {{#callable:args_values}} -->
The `args_values` function retrieves the list of argument values from a given `args` structure.
- **Inputs**:
    - `args`: A pointer to a `struct args` which contains parsed argument flags and values.
- **Control Flow**:
    - The function directly accesses the `values` member of the `args` structure and returns it.
- **Output**:
    - A pointer to the `args_value` structure, which represents the list of argument values associated with the given `args`.


---
### args_value<!-- {{#callable:args_value}} -->
The `args_value` function retrieves a pointer to the `args_value` structure at a specified index from an `args` structure, if the index is within bounds.
- **Inputs**:
    - `args`: A pointer to an `args` structure, which contains an array of `args_value` structures and a count of how many values are present.
    - `idx`: An unsigned integer representing the index of the desired `args_value` within the `args` structure's values array.
- **Control Flow**:
    - Check if the provided index `idx` is greater than or equal to the `count` of values in the `args` structure.
    - If the index is out of bounds, return `NULL`.
    - If the index is valid, return a pointer to the `args_value` at the specified index in the `values` array of the `args` structure.
- **Output**:
    - A pointer to the `args_value` structure at the specified index, or `NULL` if the index is out of bounds.


---
### args_string<!-- {{#callable:args_string}} -->
The `args_string` function retrieves the string representation of an argument at a specified index from an `args` structure.
- **Inputs**:
    - `args`: A pointer to a `struct args` which contains the arguments and their values.
    - `idx`: An unsigned integer representing the index of the argument to retrieve as a string.
- **Control Flow**:
    - Check if the provided index `idx` is greater than or equal to the count of arguments in the `args` structure.
    - If the index is out of bounds, return `NULL`.
    - Otherwise, call [`args_value_as_string`](#args_value_as_string) with the argument value at the specified index and return the result.
- **Output**:
    - Returns a `const char *` which is the string representation of the argument at the specified index, or `NULL` if the index is out of bounds.
- **Functions called**:
    - [`args_value_as_string`](#args_value_as_string)


---
### args_make_commands_now<!-- {{#callable:args_make_commands_now}} -->
The `args_make_commands_now` function prepares and executes a command list based on the provided command and arguments, handling errors and managing command references.
- **Inputs**:
    - `self`: A pointer to a `struct cmd` representing the command to be executed.
    - `item`: A pointer to a `struct cmdq_item` representing the command queue item associated with the command.
    - `idx`: An unsigned integer representing the index of the argument to be used for command preparation.
    - `expand`: An integer flag indicating whether to expand the command string during preparation.
- **Control Flow**:
    - Call [`args_make_commands_prepare`](#args_make_commands_prepare) to prepare the command state using the provided command, item, index, and expand flag.
    - Call [`args_make_commands`](#args_make_commands) with the prepared state to generate the command list, capturing any error message if the command list is NULL.
    - If the command list is NULL, log the error using `cmdq_error` and free the error message.
    - If the command list is not NULL, increment its reference count.
    - Free the command state using [`args_make_commands_free`](#args_make_commands_free).
- **Output**:
    - Returns a pointer to a `struct cmd_list`, which is the generated command list, or NULL if an error occurred.
- **Functions called**:
    - [`args_make_commands_prepare`](#args_make_commands_prepare)
    - [`args_make_commands`](#args_make_commands)
    - [`args_make_commands_free`](#args_make_commands_free)


---
### args_make_commands_prepare<!-- {{#callable:args_make_commands_prepare}} -->
The `args_make_commands_prepare` function prepares a command state by extracting and processing command arguments, setting up necessary references, and optionally expanding command strings.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being processed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
    - `idx`: An unsigned integer representing the index of the argument to process.
    - `default_command`: A constant character pointer to the default command string to use if the argument at `idx` is not present.
    - `wait`: An integer flag indicating whether to wait for the command to complete.
    - `expand`: An integer flag indicating whether to expand the command string using the target context.
- **Control Flow**:
    - Retrieve the arguments associated with the command using `cmd_get_args(self)`.
    - Get the target state and target client from the command queue item using `cmdq_get_target(item)` and `cmdq_get_target_client(item)` respectively.
    - Allocate memory for a new `args_command_state` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc).
    - Check if the index `idx` is within the range of available arguments; if so, retrieve the argument value.
    - If the argument type is `ARGS_COMMANDS`, set the command list in the state and increment its reference count, then return the state.
    - If the argument type is not `ARGS_COMMANDS`, use the argument string as the command; if `idx` is out of range, use `default_command` or trigger a fatal error if `default_command` is NULL.
    - If `expand` is true, format the command string using [`format_single_from_target`](format.c.driver.md#format_single_from_target); otherwise, duplicate the command string using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - Log the command string for debugging purposes.
    - If `wait` is true, set the `pi.item` in the state to the current item.
    - Retrieve the source file and line number using [`cmd_get_source`](cmd.c.driver.md#cmd_get_source) and store them in the state.
    - If a source file is present, duplicate it using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and store it in the state.
    - Set the target client in the state and increment its reference count if it is not NULL.
    - Copy the target find state into the state using [`cmd_find_copy_state`](cmd-find.c.driver.md#cmd_find_copy_state).
    - Return the prepared `args_command_state` structure.
- **Output**:
    - Returns a pointer to an `args_command_state` structure containing the prepared command state, including command strings, command lists, and associated references.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`format_single_from_target`](format.c.driver.md#format_single_from_target)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`cmd_get_source`](cmd.c.driver.md#cmd_get_source)
    - [`cmd_find_copy_state`](cmd-find.c.driver.md#cmd_find_copy_state)


---
### args_make_commands<!-- {{#callable:args_make_commands}} -->
The `args_make_commands` function generates a list of commands from a given command state and argument vector, handling command template replacements and parsing.
- **Inputs**:
    - `state`: A pointer to an `args_command_state` structure containing the initial command state and parsing information.
    - `argc`: An integer representing the number of arguments in the `argv` array.
    - `argv`: An array of strings representing the arguments to be used for command template replacements.
    - `error`: A pointer to a string where an error message will be stored if command parsing fails.
- **Control Flow**:
    - Check if `state->cmdlist` is not NULL; if true and `argc` is 0, return `state->cmdlist`; otherwise, return a copy of `state->cmdlist` with arguments replaced.
    - Duplicate the command string from `state->cmd` using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - Log the initial command and arguments for debugging purposes.
    - Iterate over each argument in `argv`, replacing placeholders in the command template with the current argument using [`cmd_template_replace`](cmd.c.driver.md#cmd_template_replace).
    - Log each replacement for debugging and update the command string.
    - Parse the final command string using `cmd_parse_from_string`.
    - Free the command string after parsing.
    - Check the parsing result status; if it's an error, set the error message and return NULL; if successful, return the parsed command list.
    - If the parse result status is invalid, call `fatalx` to handle the error.
- **Output**:
    - Returns a pointer to a `cmd_list` structure containing the parsed commands, or NULL if parsing fails.
- **Functions called**:
    - [`cmd_list_copy`](cmd.c.driver.md#cmd_list_copy)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`cmd_template_replace`](cmd.c.driver.md#cmd_template_replace)


---
### args_make_commands_free<!-- {{#callable:args_make_commands_free}} -->
The `args_make_commands_free` function deallocates memory and resources associated with a given `args_command_state` structure.
- **Inputs**:
    - `state`: A pointer to an `args_command_state` structure that holds command-related data to be freed.
- **Control Flow**:
    - Check if `state->cmdlist` is not NULL; if so, call [`cmd_list_free`](cmd.c.driver.md#cmd_list_free) to free the command list.
    - Check if `state->pi.c` is not NULL; if so, call [`server_client_unref`](server-client.c.driver.md#server_client_unref) to unreference the client.
    - Free the memory allocated for `state->pi.file`.
    - Free the memory allocated for `state->cmd`.
    - Free the memory allocated for the `state` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided `args_command_state` structure.
- **Functions called**:
    - [`cmd_list_free`](cmd.c.driver.md#cmd_list_free)
    - [`server_client_unref`](server-client.c.driver.md#server_client_unref)


---
### args_make_commands_get_command<!-- {{#callable:args_make_commands_get_command}} -->
The function `args_make_commands_get_command` retrieves the name of the first command from a command list or extracts a command from a string in a command state.
- **Inputs**:
    - `state`: A pointer to a `struct args_command_state` which contains the command list and command string.
- **Control Flow**:
    - Check if `state->cmdlist` is not NULL.
    - If `state->cmdlist` is not NULL, retrieve the first command using [`cmd_list_first`](cmd.c.driver.md#cmd_list_first).
    - If the first command is NULL, return an empty string duplicated using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - If the first command is not NULL, return the name of the command entry duplicated using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - If `state->cmdlist` is NULL, find the first occurrence of a space or comma in `state->cmd` using `strcspn`.
    - Format the substring of `state->cmd` up to the found position into a new string `s` using [`xasprintf`](xmalloc.c.driver.md#xasprintf).
    - Return the formatted string `s`.
- **Output**:
    - A dynamically allocated string containing the name of the first command or a substring of the command string.
- **Functions called**:
    - [`cmd_list_first`](cmd.c.driver.md#cmd_list_first)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`cmd_get_entry`](cmd.c.driver.md#cmd_get_entry)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### args_first_value<!-- {{#callable:args_first_value}} -->
The `args_first_value` function retrieves the first value associated with a specified flag from a given set of arguments.
- **Inputs**:
    - `args`: A pointer to a `struct args` which represents a set of parsed argument flags and values.
    - `flag`: An `u_char` representing the specific flag for which the first value is to be retrieved.
- **Control Flow**:
    - Call [`args_find`](#args_find) to locate the `args_entry` associated with the given flag within the `args` structure.
    - If [`args_find`](#args_find) returns NULL, indicating the flag is not present, return NULL.
    - If the `args_entry` is found, return the first value in the `values` list of the `args_entry` using `TAILQ_FIRST`.
- **Output**:
    - Returns a pointer to the first `args_value` associated with the specified flag, or NULL if the flag is not present or has no values.
- **Functions called**:
    - [`args_find`](#args_find)


---
### args_next_value<!-- {{#callable:args_next_value}} -->
The `args_next_value` function retrieves the next `args_value` in a linked list of argument values.
- **Inputs**:
    - `value`: A pointer to the current `args_value` structure from which the next value is to be retrieved.
- **Control Flow**:
    - The function uses the `TAILQ_NEXT` macro to access the next element in the linked list, using the `entry` field as the link.
- **Output**:
    - A pointer to the next `args_value` structure in the list, or `NULL` if there is no next value.


---
### args_strtonum<!-- {{#callable:args_strtonum}} -->
The `args_strtonum` function converts a string argument associated with a specific flag in an `args` structure to a number within a specified range, returning an error message if the conversion fails.
- **Inputs**:
    - `args`: A pointer to an `args` structure that contains parsed command-line arguments.
    - `flag`: An unsigned character representing the specific flag whose associated argument is to be converted to a number.
    - `minval`: The minimum allowable value for the conversion result.
    - `maxval`: The maximum allowable value for the conversion result.
    - `cause`: A pointer to a character pointer where an error message will be stored if the conversion fails.
- **Control Flow**:
    - The function attempts to find the `args_entry` associated with the given `flag` using [`args_find`](#args_find).
    - If the entry is not found, it sets `*cause` to "missing" and returns 0.
    - It retrieves the last `args_value` from the entry's values list.
    - If the value is not found, is not of type `ARGS_STRING`, or its string is NULL, it sets `*cause` to "missing" and returns 0.
    - The function calls [`strtonum`](compat/strtonum.c.driver.md#strtonum) to convert the string to a number within the specified range, capturing any error in `errstr`.
    - If [`strtonum`](compat/strtonum.c.driver.md#strtonum) returns an error, it sets `*cause` to the error message and returns 0.
    - If the conversion is successful, it sets `*cause` to NULL and returns the converted number.
- **Output**:
    - Returns the converted number if successful, or 0 if an error occurs, with `*cause` set to an error message if applicable.
- **Functions called**:
    - [`args_find`](#args_find)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)


---
### args_strtonum_and_expand<!-- {{#callable:args_strtonum_and_expand}} -->
The function `args_strtonum_and_expand` converts a string argument to a number within a specified range, expanding any format strings, and returns the result or an error message.
- **Inputs**:
    - `args`: A pointer to a `struct args` which contains the parsed command-line arguments.
    - `flag`: An unsigned character representing the flag associated with the argument to be converted.
    - `minval`: The minimum allowable value for the conversion.
    - `maxval`: The maximum allowable value for the conversion.
    - `item`: A pointer to a `struct cmdq_item` used for formatting the argument string.
    - `cause`: A pointer to a character pointer where an error message will be stored if the conversion fails.
- **Control Flow**:
    - Check if the argument associated with the given flag exists using [`args_find`](#args_find); if not, set `cause` to "missing" and return 0.
    - Retrieve the last value of the argument entry; if it is not a string or is NULL, set `cause` to "missing" and return 0.
    - Format the string value using [`format_single_from_target`](format.c.driver.md#format_single_from_target) with the provided `item`.
    - Convert the formatted string to a number using [`strtonum`](compat/strtonum.c.driver.md#strtonum), ensuring it falls within `minval` and `maxval`; if an error occurs, set `cause` to the error message and return 0.
    - If no error occurs, set `cause` to NULL and return the converted number.
- **Output**:
    - Returns the converted number if successful, or 0 if an error occurs, with `cause` set to an error message if applicable.
- **Functions called**:
    - [`args_find`](#args_find)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`format_single_from_target`](format.c.driver.md#format_single_from_target)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)


---
### args_percentage<!-- {{#callable:args_percentage}} -->
The `args_percentage` function retrieves a string value associated with a specified flag from an argument structure, checks if it is a percentage, and converts it to a numerical value within specified bounds.
- **Inputs**:
    - `args`: A pointer to a `struct args` which contains the argument flags and values.
    - `flag`: An unsigned character representing the flag to search for in the arguments.
    - `minval`: A long long integer representing the minimum allowable value for the conversion.
    - `maxval`: A long long integer representing the maximum allowable value for the conversion.
    - `curval`: A long long integer representing the current value used for percentage calculations.
    - `cause`: A pointer to a character pointer used to store error messages if the function encounters an issue.
- **Control Flow**:
    - Call [`args_find`](#args_find) to locate the `args_entry` associated with the given flag in the `args` structure.
    - If the entry is not found, set `*cause` to "missing" and return 0.
    - If the entry's values are empty, set `*cause` to "empty" and return 0.
    - Retrieve the last string value from the entry's values.
    - Call [`args_string_percentage`](#args_string_percentage) with the retrieved string and other parameters to convert the string to a numerical value.
- **Output**:
    - Returns a long long integer representing the converted numerical value, or 0 if an error occurs, with `*cause` set to an appropriate error message.
- **Functions called**:
    - [`args_find`](#args_find)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`args_string_percentage`](#args_string_percentage)


---
### args_string_percentage<!-- {{#callable:args_string_percentage}} -->
The `args_string_percentage` function converts a string representing a number or a percentage into a long long integer, ensuring it falls within specified bounds, and provides error messages if the conversion fails.
- **Inputs**:
    - `value`: A string representing a number or a percentage (e.g., '50' or '50%').
    - `minval`: The minimum allowable value for the conversion.
    - `maxval`: The maximum allowable value for the conversion.
    - `curval`: The current value used as a base for percentage calculations.
    - `cause`: A pointer to a string that will store the error message if the conversion fails.
- **Control Flow**:
    - Check if the input string `value` is empty; if so, set `cause` to 'empty' and return 0.
    - Determine if the string ends with a '%' character, indicating a percentage.
    - If it is a percentage, remove the '%' character, convert the remaining string to a number between 0 and 100, and calculate the percentage of `curval`.
    - If the conversion fails, set `cause` to the error message and return 0.
    - Check if the calculated percentage value is within `minval` and `maxval`; if not, set `cause` to 'too small' or 'too large' and return 0.
    - If the string is not a percentage, convert it directly to a number within `minval` and `maxval`.
    - If the conversion fails, set `cause` to the error message and return 0.
    - If all checks pass, set `cause` to NULL and return the converted value.
- **Output**:
    - Returns the converted long long integer value if successful, or 0 if an error occurs, with `cause` set to the error message.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)


---
### args_percentage_and_expand<!-- {{#callable:args_percentage_and_expand}} -->
The function `args_percentage_and_expand` retrieves a string argument associated with a specified flag, converts it to a number which may be a percentage, and expands any format strings, returning the result within specified bounds.
- **Inputs**:
    - `args`: A pointer to a `struct args` which contains parsed command-line arguments.
    - `flag`: An unsigned character representing the flag whose associated value is to be processed.
    - `minval`: A long long integer representing the minimum allowable value for the conversion.
    - `maxval`: A long long integer representing the maximum allowable value for the conversion.
    - `curval`: A long long integer representing the current value, used as a base for percentage calculations.
    - `item`: A pointer to a `struct cmdq_item` used for format expansion.
    - `cause`: A pointer to a string where an error message will be stored if the function encounters an error.
- **Control Flow**:
    - The function first attempts to find the `args_entry` associated with the given `flag` using [`args_find`](#args_find).
    - If the entry is not found, it sets `*cause` to "missing" and returns 0.
    - If the entry's values are empty, it sets `*cause` to "empty" and returns 0.
    - It retrieves the last value associated with the entry's values list.
    - The function then calls [`args_string_percentage_and_expand`](#args_string_percentage_and_expand) with the retrieved value and other parameters to perform the conversion and expansion.
- **Output**:
    - The function returns a long long integer representing the converted and expanded value, or 0 if an error occurs, with `*cause` set to an appropriate error message.
- **Functions called**:
    - [`args_find`](#args_find)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`args_string_percentage_and_expand`](#args_string_percentage_and_expand)


---
### args_string_percentage_and_expand<!-- {{#callable:args_string_percentage_and_expand}} -->
The function `args_string_percentage_and_expand` converts a string to a number, potentially interpreting it as a percentage, and expands any format strings, ensuring the result is within specified bounds.
- **Inputs**:
    - `value`: A string representing a number or a percentage, potentially containing format strings.
    - `minval`: The minimum allowable value for the conversion result.
    - `maxval`: The maximum allowable value for the conversion result.
    - `curval`: The current value used as a reference for percentage calculations.
    - `item`: A `cmdq_item` structure used for format string expansion.
    - `cause`: A pointer to a string that will be set to an error message if the conversion fails.
- **Control Flow**:
    - Determine the length of the input string `value`.
    - Check if the last character of `value` is a '%'.
    - If it is a percentage, duplicate the string without the '%' and expand any format strings using [`format_single_from_target`](format.c.driver.md#format_single_from_target).
    - Convert the expanded string to a number between 0 and 100 using [`strtonum`](compat/strtonum.c.driver.md#strtonum).
    - If conversion fails, set `cause` to the error message and return 0.
    - Calculate the percentage of `curval` and check if it falls within `minval` and `maxval`.
    - If the percentage is out of bounds, set `cause` to an appropriate error message and return 0.
    - If `value` is not a percentage, expand any format strings and convert it to a number between `minval` and `maxval`.
    - If conversion fails, set `cause` to the error message and return 0.
    - If successful, set `cause` to NULL and return the converted number.
- **Output**:
    - Returns the converted number as a `long long`, or 0 if an error occurs, with `cause` set to an error message if applicable.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`format_single_from_target`](format.c.driver.md#format_single_from_target)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)


