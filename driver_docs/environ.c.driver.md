# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides functionality for managing environment variables within the tmux application. The code defines a set of functions to create, manipulate, and manage a custom environment structure, which is used to handle environment variables in a controlled manner. The primary data structure used is a red-black tree, which efficiently stores and retrieves environment variables. The file includes functions to create and free environments, set and unset variables, copy environments, and push the custom environment into the real system environment after a process fork.

The code is organized around the `environ` structure, which represents a collection of environment variables. Key functions include [`environ_create`](#environ_create) for initializing a new environment, [`environ_set`](#environ_set) and [`environ_unset`](#environ_unset) for modifying variables, and [`environ_push`](#environ_push) for applying the environment to the system. The file also includes utility functions like [`environ_log`](#environ_log) for debugging and [`environ_for_session`](#environ_for_session) for creating an initial environment for a new tmux session. This code is not a standalone executable but rather a component of the tmux codebase, intended to be used internally to manage environment variables in a way that is consistent and isolated from the system environment, enhancing the modularity and reliability of the tmux application.
# Imports and Dependencies

---
- `sys/types.h`
- `fnmatch.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `tmux.h`


# Functions

---
### environ_cmp<!-- {{#callable:environ_cmp}} -->
The `environ_cmp` function compares two environment entries by their names using the `strcmp` function.
- **Inputs**:
    - `envent1`: A pointer to the first `environ_entry` structure to be compared.
    - `envent2`: A pointer to the second `environ_entry` structure to be compared.
- **Control Flow**:
    - The function takes two pointers to `environ_entry` structures as input.
    - It calls the `strcmp` function to compare the `name` fields of the two structures.
    - The result of `strcmp` is returned, which indicates the lexicographical order of the names.
- **Output**:
    - The function returns an integer that is less than, equal to, or greater than zero if the name of `envent1` is found, respectively, to be less than, to match, or be greater than the name of `envent2`.


---
### environ_create<!-- {{#callable:environ_create}} -->
The `environ_create` function initializes and returns a new environment structure for managing environment variables.
- **Inputs**:
    - None
- **Control Flow**:
    - Allocate memory for a new `environ` structure using `xcalloc`, initializing it to zero.
    - Initialize the red-black tree within the `environ` structure using `RB_INIT`.
    - Return the pointer to the newly created `environ` structure.
- **Output**:
    - A pointer to a newly allocated and initialized `environ` structure is returned.


---
### environ_free<!-- {{#callable:environ_free}} -->
The `environ_free` function deallocates memory associated with an environment structure and its entries.
- **Inputs**:
    - `env`: A pointer to a `struct environ` which represents the environment to be freed.
- **Control Flow**:
    - Iterates over each entry in the environment using `RB_FOREACH_SAFE`, which allows safe removal of entries during iteration.
    - For each entry, it removes the entry from the environment tree using `RB_REMOVE`.
    - Frees the memory allocated for the entry's name, value, and the entry itself.
    - After all entries are removed and freed, it frees the memory allocated for the environment structure itself.
- **Output**:
    - The function does not return any value; it performs memory deallocation for the given environment structure and its entries.


---
### environ_first<!-- {{#callable:environ_first}} -->
The `environ_first` function retrieves the first entry in a red-black tree representing an environment.
- **Inputs**:
    - `env`: A pointer to an `environ` structure, which is a red-black tree of environment entries.
- **Control Flow**:
    - The function calls `RB_MIN` macro with the `environ` type and the provided `env` pointer to find the minimum (first) entry in the red-black tree.
- **Output**:
    - A pointer to the first `environ_entry` in the red-black tree, or `NULL` if the tree is empty.


---
### environ_next<!-- {{#callable:environ_next}} -->
The `environ_next` function retrieves the next entry in a red-black tree of environment entries.
- **Inputs**:
    - `envent`: A pointer to the current `environ_entry` from which the next entry is to be retrieved.
- **Control Flow**:
    - The function calls `RB_NEXT` macro with the `environ` tree, `env` as the tree head, and `envent` as the current node to find the next node in the tree.
    - The result of `RB_NEXT` is returned, which is the next `environ_entry` in the tree or `NULL` if there is no next entry.
- **Output**:
    - A pointer to the next `environ_entry` in the red-black tree, or `NULL` if there is no next entry.


---
### environ_copy<!-- {{#callable:environ_copy}} -->
The `environ_copy` function copies environment variables from a source environment to a destination environment.
- **Inputs**:
    - `srcenv`: A pointer to the source environment structure from which variables are copied.
    - `dstenv`: A pointer to the destination environment structure to which variables are copied.
- **Control Flow**:
    - Iterates over each environment entry in the source environment `srcenv` using the `RB_FOREACH` macro.
    - For each entry, checks if the `value` is `NULL`.
    - If `value` is `NULL`, calls [`environ_clear`](#environ_clear) to remove the variable from the destination environment `dstenv`.
    - If `value` is not `NULL`, calls [`environ_set`](#environ_set) to set the variable in the destination environment `dstenv` with the same name, flags, and value as in the source environment.
- **Output**:
    - The function does not return a value; it modifies the destination environment `dstenv` in place.
- **Functions called**:
    - [`environ_clear`](#environ_clear)
    - [`environ_set`](#environ_set)


---
### environ_find<!-- {{#callable:environ_find}} -->
The `environ_find` function searches for an environment variable by name within a given environment structure and returns a pointer to the corresponding `environ_entry` if found.
- **Inputs**:
    - `env`: A pointer to the `environ` structure representing the environment in which to search for the variable.
    - `name`: A constant character pointer representing the name of the environment variable to find.
- **Control Flow**:
    - A temporary `environ_entry` structure `envent` is created and its `name` field is set to the input `name` cast to a non-const character pointer.
    - The function calls `RB_FIND` with the `environ` tree, the `env` structure, and the address of `envent` to search for a matching entry.
    - The result of `RB_FIND`, which is a pointer to the found `environ_entry` or `NULL` if not found, is returned.
- **Output**:
    - A pointer to the `environ_entry` structure corresponding to the found environment variable, or `NULL` if the variable is not found.


---
### environ_set<!-- {{#callable:environ_set}} -->
The `environ_set` function sets or updates an environment variable in a given environment structure.
- **Inputs**:
    - `env`: A pointer to the `environ` structure where the environment variable should be set or updated.
    - `name`: A string representing the name of the environment variable to set or update.
    - `flags`: An integer representing flags associated with the environment variable.
    - `fmt`: A format string for the value of the environment variable, followed by additional arguments as needed.
- **Control Flow**:
    - Initialize a variable argument list using `va_start` with the format string `fmt`.
    - Check if an environment entry with the given `name` already exists using [`environ_find`](#environ_find).
    - If the entry exists, update its `flags`, free its current `value`, and set a new `value` using `xvasprintf` with the provided format and arguments.
    - If the entry does not exist, allocate memory for a new `environ_entry`, set its `name` and `flags`, allocate and set its `value` using `xvasprintf`, and insert it into the environment using `RB_INSERT`.
    - End the variable argument list using `va_end`.
- **Output**:
    - The function does not return a value; it modifies the environment structure in place by setting or updating an environment variable.
- **Functions called**:
    - [`environ_find`](#environ_find)


---
### environ_clear<!-- {{#callable:environ_clear}} -->
The `environ_clear` function clears the value of a specified environment variable in a given environment structure, or creates a new entry if it does not exist.
- **Inputs**:
    - `env`: A pointer to the `environ` structure representing the environment where the variable is to be cleared.
    - `name`: A constant character pointer representing the name of the environment variable to be cleared.
- **Control Flow**:
    - The function attempts to find the environment entry corresponding to the given name using [`environ_find`](#environ_find).
    - If the entry is found, it frees the memory allocated for the entry's value and sets the value to NULL.
    - If the entry is not found, it allocates memory for a new `environ_entry`, duplicates the name, sets the flags to 0, sets the value to NULL, and inserts the new entry into the environment using `RB_INSERT`.
- **Output**:
    - The function does not return a value; it modifies the environment structure in place.
- **Functions called**:
    - [`environ_find`](#environ_find)


---
### environ_put<!-- {{#callable:environ_put}} -->
The `environ_put` function sets an environment variable in a given environment structure from a string in the format NAME=VALUE.
- **Inputs**:
    - `env`: A pointer to the `environ` structure where the environment variable should be set.
    - `var`: A string containing the environment variable in the format NAME=VALUE.
    - `flags`: An integer representing flags that may affect how the environment variable is set.
- **Control Flow**:
    - The function searches for the '=' character in the `var` string to separate the name and value of the environment variable.
    - If the '=' character is not found, the function returns immediately without making any changes.
    - If the '=' character is found, the function duplicates the `var` string and terminates the name part at the '=' character.
    - The function then calls [`environ_set`](#environ_set) to set the environment variable in the `env` structure using the extracted name and value.
    - Finally, the function frees the duplicated name string to avoid memory leaks.
- **Output**:
    - The function does not return any value; it modifies the `env` structure by setting the specified environment variable.
- **Functions called**:
    - [`environ_set`](#environ_set)


---
### environ_unset<!-- {{#callable:environ_unset}} -->
The `environ_unset` function removes an environment variable from a given environment structure.
- **Inputs**:
    - `env`: A pointer to the `environ` structure from which the environment variable should be removed.
    - `name`: A constant character pointer representing the name of the environment variable to be removed.
- **Control Flow**:
    - The function first attempts to find the environment entry corresponding to the given name using [`environ_find`](#environ_find).
    - If the entry is not found (i.e., [`environ_find`](#environ_find) returns NULL), the function returns immediately without making any changes.
    - If the entry is found, it is removed from the environment tree using `RB_REMOVE`.
    - The memory allocated for the entry's name, value, and the entry itself is freed using `free`.
- **Output**:
    - The function does not return any value; it performs the operation of removing an environment variable and freeing associated resources.
- **Functions called**:
    - [`environ_find`](#environ_find)


---
### environ_update<!-- {{#callable:environ_update}} -->
The `environ_update` function updates the destination environment with variables from the source environment based on patterns specified in the options.
- **Inputs**:
    - `oo`: A pointer to a struct options, which contains configuration options including the 'update-environment' array.
    - `src`: A pointer to the source environment from which variables are copied.
    - `dst`: A pointer to the destination environment where variables are updated or cleared.
- **Control Flow**:
    - Retrieve the 'update-environment' option from the options structure `oo`.
    - If the option is not found, exit the function.
    - Iterate over each item in the 'update-environment' array.
    - For each item, iterate over each entry in the source environment `src`.
    - Check if the source environment entry name matches the current pattern using `fnmatch`.
    - If a match is found, set the corresponding variable in the destination environment `dst` with the source value.
    - If no match is found for a pattern, clear the corresponding variable in the destination environment `dst`.
- **Output**:
    - The function does not return a value; it modifies the destination environment `dst` in place by setting or clearing variables based on the source environment `src` and the patterns specified in the options.
- **Functions called**:
    - [`environ_set`](#environ_set)
    - [`environ_clear`](#environ_clear)


---
### environ_push<!-- {{#callable:environ_push}} -->
The `environ_push` function updates the real environment variables with those from a given environment structure, excluding hidden variables.
- **Inputs**:
    - `env`: A pointer to an `environ` structure containing environment entries to be pushed to the real environment.
- **Control Flow**:
    - Allocate memory for a new environment structure using `xcalloc`.
    - Iterate over each `environ_entry` in the provided `env` using `RB_FOREACH`.
    - For each entry, check if the `value` is not NULL, the `name` is not empty, and the `ENVIRON_HIDDEN` flag is not set.
    - If all conditions are met, use `setenv` to set the environment variable in the real environment.
- **Output**:
    - The function does not return a value; it modifies the real environment variables directly.


---
### environ_log<!-- {{#callable:environ_log}} -->
The `environ_log` function logs all environment variables from a given environment structure with a specified prefix.
- **Inputs**:
    - `env`: A pointer to a `struct environ`, which is a data structure representing a set of environment variables.
    - `fmt`: A format string used to create a prefix for each log entry, followed by a variable number of arguments.
- **Control Flow**:
    - Initialize a variable argument list using `va_start` with the format string `fmt`.
    - Use `vasprintf` to create a formatted string `prefix` from the format string and arguments.
    - Iterate over each environment entry in the `env` structure using `RB_FOREACH`.
    - For each entry, if the `value` is not NULL and the `name` is not an empty string, log the entry using `log_debug` with the format `"%s%s=%s"`, where `%s` is replaced by `prefix`, `envent->name`, and `envent->value`.
    - Free the allocated memory for `prefix` using `free`.
- **Output**:
    - The function does not return a value; it performs logging as a side effect.


---
### environ_for_session<!-- {{#callable:environ_for_session}} -->
The `environ_for_session` function creates and initializes an environment structure for a session, optionally setting terminal-related variables and clearing systemd-related variables.
- **Inputs**:
    - `s`: A pointer to a `session` structure, which may contain its own environment to be copied into the new environment.
    - `no_TERM`: An integer flag indicating whether terminal-related environment variables should be set (0 to set, non-zero to skip).
- **Control Flow**:
    - Create a new environment structure using [`environ_create`](#environ_create).
    - Copy the global environment into the new environment using [`environ_copy`](#environ_copy).
    - If the session `s` is not NULL, copy its environment into the new environment.
    - If `no_TERM` is false (0), set the `TERM`, `TERM_PROGRAM`, and `TERM_PROGRAM_VERSION` environment variables using [`environ_set`](#environ_set).
    - If compiled with systemd support (`HAVE_SYSTEMD`), clear the `LISTEN_PID`, `LISTEN_FDS`, and `LISTEN_FDNAMES` environment variables using [`environ_clear`](#environ_clear).
    - Determine the session index `idx` from `s` if it is not NULL, otherwise set `idx` to -1.
    - Set the `TMUX` environment variable with the socket path, process ID, and session index using [`environ_set`](#environ_set).
- **Output**:
    - Returns a pointer to the newly created and initialized `environ` structure.
- **Functions called**:
    - [`environ_create`](#environ_create)
    - [`environ_copy`](#environ_copy)
    - [`environ_set`](#environ_set)
    - [`environ_clear`](#environ_clear)


# Function Declarations (Public API)

---
### environ_cmp<!-- {{#callable_declaration:environ_cmp}} -->
Compare two environment entries by their names.
- **Description**: Use this function to determine the lexicographical order of two environment entries based on their names. It is typically used in contexts where environment entries need to be sorted or compared, such as in a red-black tree. The function assumes that both input parameters are valid pointers to `environ_entry` structures with non-null `name` fields.
- **Inputs**:
    - `envent1`: A pointer to the first `environ_entry` structure to compare. Must not be null and must have a valid `name` field.
    - `envent2`: A pointer to the second `environ_entry` structure to compare. Must not be null and must have a valid `name` field.
- **Output**: Returns an integer less than, equal to, or greater than zero if the name of `envent1` is found, respectively, to be less than, to match, or be greater than the name of `envent2`.
- **See also**: [`environ_cmp`](#environ_cmp)  (Implementation)


