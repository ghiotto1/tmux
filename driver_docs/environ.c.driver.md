# Purpose
This C source code file is part of the tmux project and is responsible for managing environment variables within the application. It provides a set of functions to create, manipulate, and manage a custom environment structure, which is used to handle environment variables in a controlled manner. The code defines a red-black tree data structure to store environment entries, allowing efficient insertion, deletion, and lookup operations. Key functions include creating and freeing environment structures, setting and unsetting environment variables, copying environments, and updating environments based on specific patterns. The code also includes functionality to push the custom environment into the real environment after a process fork, ensuring that child processes inherit the correct environment settings.

The file is not an executable but rather a component of the tmux application, likely intended to be included and used by other parts of the program. It does not define public APIs or external interfaces directly but provides internal functionality crucial for managing environment variables within tmux sessions. The code also includes provisions for logging environment variables and creating initial environments for new child processes, ensuring that the tmux environment is correctly set up and maintained across different sessions and processes.
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
    - The function calls `strcmp` with the `name` fields of `envent1` and `envent2` as arguments.
    - The result of `strcmp` is returned directly.
- **Output**:
    - The function returns an integer less than, equal to, or greater than zero if the name of `envent1` is found, respectively, to be less than, to match, or be greater than the name of `envent2`.


---
### environ_create<!-- {{#callable:environ_create}} -->
The `environ_create` function initializes and returns a new environment structure for managing environment variables.
- **Inputs**:
    - None
- **Control Flow**:
    - Allocate memory for a new `environ` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc), which initializes the allocated memory to zero.
    - Initialize the newly allocated `environ` structure as a red-black tree using `RB_INIT`.
    - Return the pointer to the newly created and initialized `environ` structure.
- **Output**:
    - A pointer to a newly created and initialized `environ` structure is returned.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


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
    - The function does not return any value; it performs memory deallocation for the given environment and its entries.


---
### environ_first<!-- {{#callable:environ_first}} -->
The `environ_first` function retrieves the first entry in a red-black tree representing an environment.
- **Inputs**:
    - `env`: A pointer to a struct `environ`, which is a red-black tree containing environment entries.
- **Control Flow**:
    - The function calls `RB_MIN` macro with the `environ` tree type and the provided `env` pointer to find the minimum (first) entry in the tree.
- **Output**:
    - A pointer to the first `environ_entry` in the red-black tree, or NULL if the tree is empty.


---
### environ_next<!-- {{#callable:environ_next}} -->
The `environ_next` function retrieves the next entry in a red-black tree of environment entries.
- **Inputs**:
    - `envent`: A pointer to the current `environ_entry` from which the next entry is to be retrieved.
- **Control Flow**:
    - The function calls the macro `RB_NEXT` with the parameters `environ`, `env`, and `envent` to find the next entry in the red-black tree.
    - The macro `RB_NEXT` is used to traverse the red-black tree structure to find the next node after the given `envent`.
- **Output**:
    - The function returns a pointer to the next `environ_entry` in the red-black tree, or `NULL` if there is no next entry.


---
### environ_copy<!-- {{#callable:environ_copy}} -->
The `environ_copy` function copies environment variables from a source environment to a destination environment.
- **Inputs**:
    - `srcenv`: A pointer to the source environment structure from which variables are copied.
    - `dstenv`: A pointer to the destination environment structure where variables are copied to.
- **Control Flow**:
    - Iterate over each environment entry in the source environment `srcenv` using the `RB_FOREACH` macro.
    - For each entry, check if the `value` is `NULL`.
    - If `value` is `NULL`, call [`environ_clear`](#environ_clear) to remove the variable from the destination environment `dstenv`.
    - If `value` is not `NULL`, call [`environ_set`](#environ_set) to set the variable in the destination environment `dstenv` with the same name, flags, and value.
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
    - A temporary `environ_entry` structure `envent` is created and its `name` field is set to the input `name` argument cast to a `char *`.
    - The function calls `RB_FIND` with the `environ` tree, the `env` structure, and the address of `envent` to search for the environment variable.
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
    - Check if the environment variable `name` already exists in `env` using [`environ_find`](#environ_find).
    - If the variable exists, update its `flags`, free its current value, and set a new value using [`xvasprintf`](xmalloc.c.driver.md#xvasprintf).
    - If the variable does not exist, allocate memory for a new `environ_entry`, set its `name`, `flags`, and value using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and [`xvasprintf`](xmalloc.c.driver.md#xvasprintf), and insert it into the environment using `RB_INSERT`.
    - End the variable argument list using `va_end`.
- **Output**:
    - The function does not return a value; it modifies the environment structure `env` in place.
- **Functions called**:
    - [`environ_find`](#environ_find)
    - [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


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
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### environ_put<!-- {{#callable:environ_put}} -->
The `environ_put` function sets an environment variable in a given environment structure from a string in the format 'NAME=VALUE'.
- **Inputs**:
    - `env`: A pointer to the `environ` structure where the environment variable should be set.
    - `var`: A string containing the environment variable in the format 'NAME=VALUE'.
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
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
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
    - If the entry is found, it is removed from the red-black tree using `RB_REMOVE`.
    - The memory allocated for the entry's name, value, and the entry itself is freed using `free`.
- **Output**:
    - The function does not return any value; it performs the operation of removing an environment variable from the environment structure.
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
    - Check if the entry's name matches the current pattern using `fnmatch`.
    - If a match is found, set the corresponding variable in the destination environment `dst` with the same value.
    - If no match is found for a pattern, clear the corresponding variable in the destination environment `dst`.
    - Continue to the next item in the 'update-environment' array.
- **Output**:
    - The function does not return a value; it modifies the destination environment `dst` in place.
- **Functions called**:
    - [`options_get`](options.c.driver.md#options_get)
    - [`options_array_first`](options.c.driver.md#options_array_first)
    - [`options_array_item_value`](options.c.driver.md#options_array_item_value)
    - [`environ_set`](#environ_set)
    - [`environ_clear`](#environ_clear)
    - [`options_array_next`](options.c.driver.md#options_array_next)


---
### environ_push<!-- {{#callable:environ_push}} -->
The `environ_push` function updates the real environment variables with those from a given environment structure, excluding hidden variables.
- **Inputs**:
    - `env`: A pointer to a `struct environ` which is a red-black tree containing environment entries to be pushed to the real environment.
- **Control Flow**:
    - Allocate memory for a new environment structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc).
    - Iterate over each `environ_entry` in the provided `env` using `RB_FOREACH`.
    - For each entry, check if the `value` is not NULL, the `name` is not an empty string, and the `ENVIRON_HIDDEN` flag is not set.
    - If all conditions are met, use [`setenv`](compat/setenv.c.driver.md#setenv) to set the environment variable in the real environment with the entry's name and value, allowing overwriting of existing variables.
- **Output**:
    - The function does not return a value; it modifies the real environment variables directly.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`setenv`](compat/setenv.c.driver.md#setenv)


---
### environ_log<!-- {{#callable:environ_log}} -->
The `environ_log` function logs all environment variables in a given environment structure with a specified prefix.
- **Inputs**:
    - `env`: A pointer to a struct environ, which is a data structure representing a set of environment variables.
    - `fmt`: A format string used to create a prefix for each log entry.
    - `...`: A variable number of arguments that are used in conjunction with the format string to create the prefix.
- **Control Flow**:
    - Initialize a variable argument list using `va_start` with the format string `fmt`.
    - Use [`vasprintf`](compat/asprintf.c.driver.md#vasprintf) to format the prefix string with the provided arguments.
    - Iterate over each environment entry in the `env` structure using `RB_FOREACH`.
    - For each entry, check if the `value` is not NULL and the `name` is not an empty string.
    - If the conditions are met, log the environment variable in the format `prefixname=value` using `log_debug`.
    - Free the allocated memory for the prefix string using `free`.
- **Output**:
    - The function does not return any value; it performs logging as a side effect.
- **Functions called**:
    - [`vasprintf`](compat/asprintf.c.driver.md#vasprintf)


---
### environ_for_session<!-- {{#callable:environ_for_session}} -->
The `environ_for_session` function creates and initializes an environment structure for a given session, optionally excluding terminal-related variables.
- **Inputs**:
    - `s`: A pointer to a `session` structure, which may contain its own environment variables to be copied into the new environment.
    - `no_TERM`: An integer flag indicating whether terminal-related environment variables should be excluded (non-zero value) or included (zero value).
- **Control Flow**:
    - Create a new environment structure using [`environ_create`](#environ_create).
    - Copy global environment variables into the new environment using [`environ_copy`](#environ_copy).
    - If the session `s` is not NULL, copy its environment variables into the new environment.
    - If `no_TERM` is zero, set the `TERM`, `TERM_PROGRAM`, and `TERM_PROGRAM_VERSION` environment variables using [`environ_set`](#environ_set).
    - If compiled with systemd support, clear the `LISTEN_PID`, `LISTEN_FDS`, and `LISTEN_FDNAMES` environment variables using [`environ_clear`](#environ_clear).
    - Determine the session index `idx` based on whether `s` is NULL or not, and set the `TMUX` environment variable using [`environ_set`](#environ_set).
    - Return the newly created and initialized environment structure.
- **Output**:
    - A pointer to the newly created and initialized `environ` structure.
- **Functions called**:
    - [`environ_create`](#environ_create)
    - [`environ_copy`](#environ_copy)
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`environ_set`](#environ_set)
    - [`getversion`](tmux.c.driver.md#getversion)
    - [`environ_clear`](#environ_clear)


