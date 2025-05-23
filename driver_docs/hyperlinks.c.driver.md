# Purpose
This C source code file is part of the tmux project and is responsible for managing OSC 8 hyperlinks. The code provides a specialized functionality for handling hyperlinks within the tmux terminal multiplexer, allowing for the creation, storage, retrieval, and removal of hyperlinks. The code defines a system where each hyperlink, identified by a URI and an ID, is assigned a unique "inner" number. This number is used to map hyperlinks into a tree structure, facilitating efficient management and lookup operations. The code includes functions to initialize, copy, reset, and free hyperlink sets, as well as to add new hyperlinks and retrieve existing ones by their inner number.

The file is not a standalone executable but rather a component of the tmux codebase, intended to be integrated with other parts of the system. It defines internal data structures and functions for managing hyperlinks, using red-black trees and tail queues to organize and access the data efficiently. The code does not define public APIs or external interfaces, as it is designed for internal use within the tmux application. The use of static functions and data structures indicates that the functionality is encapsulated within this module, ensuring that the hyperlink management logic is self-contained and not exposed to other parts of the application or external users.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### hyperlinks_next_external_id
- **Type**: `long long`
- **Description**: The `hyperlinks_next_external_id` is a static global variable of type `long long` initialized to 1. It is used to generate unique external IDs for hyperlinks in the tmux application.
- **Use**: This variable is incremented each time a new hyperlink is stored, ensuring each hyperlink has a unique external ID.


---
### global_hyperlinks_count
- **Type**: `u_int`
- **Description**: The `global_hyperlinks_count` is a static unsigned integer that keeps track of the total number of hyperlinks currently stored in the global list of hyperlinks. It is used to ensure that the number of hyperlinks does not exceed a predefined maximum limit.
- **Use**: This variable is incremented when a new hyperlink is added and decremented when a hyperlink is removed, helping manage the lifecycle of hyperlinks in the application.


---
### global_hyperlinks
- **Type**: `struct hyperlinks_list`
- **Description**: The `global_hyperlinks` variable is a static instance of a `hyperlinks_list` structure, which is a tail queue head used to manage a list of `hyperlinks_uri` structures. It is initialized using the `TAILQ_HEAD_INITIALIZER` macro, which sets up the list to be empty initially.
- **Use**: This variable is used to store and manage a global list of hyperlink URIs, allowing for insertion, removal, and traversal of hyperlinks within the application.


# Data Structures

---
### hyperlinks_uri
- **Type**: `struct`
- **Members**:
    - `tree`: A pointer to a struct of type hyperlinks, representing the tree structure this URI belongs to.
    - `inner`: An unsigned integer representing the unique internal number assigned to each hyperlink and ID combination.
    - `internal_id`: A constant character pointer to the internal ID received from the sending application.
    - `external_id`: A constant character pointer to the external ID used by tmux to send the hyperlink to the terminal.
    - `uri`: A constant character pointer to the URI associated with the hyperlink.
    - `list_entry`: A TAILQ_ENTRY structure for linking this URI in a tail queue.
    - `by_inner_entry`: An RB_ENTRY structure for linking this URI in a red-black tree sorted by the inner number.
    - `by_uri_entry`: An RB_ENTRY structure for linking this URI in a red-black tree sorted by internal ID and URI.
- **Description**: The `hyperlinks_uri` structure is designed to manage and store information about hyperlinks within a terminal multiplexer (tmux) environment. Each instance of this structure represents a unique hyperlink, identified by a combination of an internal ID, external ID, and URI. The structure includes fields for managing these identifiers, as well as entries for organizing the hyperlinks within both a tail queue and red-black trees, allowing efficient insertion, deletion, and lookup operations. The `inner` field provides a unique identifier for each hyperlink, facilitating its management within the system.


---
### hyperlinks
- **Type**: `struct`
- **Members**:
    - `next_inner`: An unsigned integer representing the next available inner number for a hyperlink.
    - `by_inner`: A red-black tree structure that organizes hyperlinks by their inner numbers.
    - `by_uri`: A red-black tree structure that organizes hyperlinks by their URI and internal ID.
    - `references`: An unsigned integer that tracks the number of references to this hyperlink structure.
- **Description**: The `hyperlinks` structure is designed to manage and organize hyperlinks within a system, specifically for use in terminal multiplexers like tmux. It maintains a collection of hyperlinks, each identified by a unique 'inner' number, and organizes them using two red-black trees: one sorted by the inner number and another by URI and internal ID. This structure also keeps track of the next available inner number and the number of references to the structure, allowing for efficient management and retrieval of hyperlink data.


# Functions

---
### hyperlinks_by_uri_cmp<!-- {{#callable:hyperlinks_by_uri_cmp}} -->
The `hyperlinks_by_uri_cmp` function compares two `hyperlinks_uri` structures based on their internal IDs and URIs to determine their order in a red-black tree.
- **Inputs**:
    - `left`: A pointer to the first `hyperlinks_uri` structure to be compared.
    - `right`: A pointer to the second `hyperlinks_uri` structure to be compared.
- **Control Flow**:
    - Check if either `left` or `right` has an empty `internal_id` indicating an anonymous URI.
    - If `left` is not anonymous and `right` is, return -1 to indicate `left` should come before `right`.
    - If `right` is not anonymous and `left` is, return 1 to indicate `right` should come before `left`.
    - If both are anonymous, compare their `inner` values and return the difference to ensure uniqueness.
    - If neither is anonymous, compare their `internal_id` strings using `strcmp`.
    - If the `internal_id` comparison is non-zero, return the result of `strcmp`.
    - If the `internal_id` comparison is zero, compare their `uri` strings using `strcmp` and return the result.
- **Output**:
    - An integer indicating the order of the two `hyperlinks_uri` structures: negative if `left` should come before `right`, positive if `right` should come before `left`, and zero if they are considered equal.


---
### hyperlinks_by_inner_cmp<!-- {{#callable:hyperlinks_by_inner_cmp}} -->
The function `hyperlinks_by_inner_cmp` compares two `hyperlinks_uri` structures based on their `inner` field values.
- **Inputs**:
    - `left`: A pointer to the first `hyperlinks_uri` structure to be compared.
    - `right`: A pointer to the second `hyperlinks_uri` structure to be compared.
- **Control Flow**:
    - The function calculates the difference between the `inner` field of the `left` and `right` `hyperlinks_uri` structures.
    - It returns the result of this subtraction, which indicates the relative ordering of the two structures based on their `inner` values.
- **Output**:
    - The function returns an integer that is the result of subtracting the `inner` value of the `right` structure from the `inner` value of the `left` structure, which is used to determine their order in a red-black tree.


---
### hyperlinks_remove<!-- {{#callable:hyperlinks_remove}} -->
The `hyperlinks_remove` function removes a hyperlink from global and local data structures and frees its associated memory.
- **Inputs**:
    - `hlu`: A pointer to a `struct hyperlinks_uri` representing the hyperlink to be removed.
- **Control Flow**:
    - Retrieve the `hyperlinks` structure from the `tree` member of the `hlu` parameter.
    - Remove the `hlu` entry from the global `TAILQ` list `global_hyperlinks` and decrement the `global_hyperlinks_count`.
    - Remove the `hlu` entry from the `by_inner` and `by_uri` red-black trees of the `hyperlinks` structure.
    - Free the memory allocated for the `internal_id`, `external_id`, and `uri` strings of the `hlu`.
    - Free the memory allocated for the `hlu` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided hyperlink structure.


---
### hyperlinks_put<!-- {{#callable:hyperlinks_put}} -->
The `hyperlinks_put` function stores a new hyperlink in a data structure or returns an existing one if it already exists, ensuring each hyperlink is unique by its URI and internal ID.
- **Inputs**:
    - `hl`: A pointer to a `struct hyperlinks` which contains the data structures for storing hyperlinks.
    - `uri_in`: A constant character pointer representing the URI of the hyperlink to be stored.
    - `internal_id_in`: A constant character pointer representing the internal ID associated with the URI, which can be NULL for anonymous URIs.
- **Control Flow**:
    - Check if `internal_id_in` is NULL and set it to an empty string if so, to handle anonymous URIs.
    - Convert `uri_in` and `internal_id_in` to UTF-8 encoded strings with special character handling using `utf8_stravis`.
    - If `internal_id_in` is not empty, search for an existing hyperlink in the `by_uri` tree using the URI and internal ID.
    - If a matching hyperlink is found, free the allocated memory for `uri` and `internal_id`, and return the existing hyperlink's inner number.
    - If no match is found, generate a new external ID using a global counter and format it as a string.
    - Allocate memory for a new `hyperlinks_uri` structure and initialize it with the URI, internal ID, external ID, and a new inner number.
    - Insert the new hyperlink into both the `by_uri` and `by_inner` trees of the `struct hyperlinks`.
    - Add the new hyperlink to the global list of hyperlinks and check if the count exceeds the maximum allowed, removing the oldest hyperlink if necessary.
    - Return the inner number of the newly added or found hyperlink.
- **Output**:
    - Returns the inner number of the hyperlink, which is a unique identifier for the hyperlink within the data structure.
- **Functions called**:
    - [`hyperlinks_remove`](#hyperlinks_remove)


---
### hyperlinks_get<!-- {{#callable:hyperlinks_get}} -->
The `hyperlinks_get` function retrieves a hyperlink's URI, internal ID, and external ID based on a given inner number from a tree structure.
- **Inputs**:
    - `hl`: A pointer to a `struct hyperlinks` which contains the tree structures for storing hyperlinks.
    - `inner`: An unsigned integer representing the inner number used to identify the hyperlink within the tree.
    - `uri_out`: A pointer to a constant character pointer where the URI of the found hyperlink will be stored.
    - `internal_id_out`: A pointer to a constant character pointer where the internal ID of the found hyperlink will be stored, if not NULL.
    - `external_id_out`: A pointer to a constant character pointer where the external ID of the found hyperlink will be stored, if not NULL.
- **Control Flow**:
    - A `struct hyperlinks_uri` named `find` is initialized with the `inner` value to search for the hyperlink.
    - The function uses `RB_FIND` to search the `by_inner` tree in `hl` for a matching hyperlink using the `find` structure.
    - If no matching hyperlink is found (`hlu` is NULL), the function returns 0 indicating failure.
    - If `internal_id_out` is not NULL, the internal ID of the found hyperlink is assigned to it.
    - If `external_id_out` is not NULL, the external ID of the found hyperlink is assigned to it.
    - The URI of the found hyperlink is assigned to `uri_out`.
    - The function returns 1 indicating success.
- **Output**:
    - Returns 1 if a hyperlink with the specified inner number is found, otherwise returns 0.


---
### hyperlinks_init<!-- {{#callable:hyperlinks_init}} -->
The `hyperlinks_init` function initializes and returns a new `hyperlinks` structure with default values.
- **Inputs**:
    - None
- **Control Flow**:
    - Allocate memory for a new `hyperlinks` structure using `xcalloc` and assign it to `hl`.
    - Set the `next_inner` field of `hl` to 1, indicating the starting point for inner hyperlink numbers.
    - Initialize the `by_uri` and `by_inner` red-black trees within `hl` using `RB_INIT`.
    - Set the `references` field of `hl` to 1, indicating the initial reference count.
    - Return the initialized `hl` structure.
- **Output**:
    - The function returns a pointer to a newly initialized `hyperlinks` structure.


---
### hyperlinks_copy<!-- {{#callable:hyperlinks_copy}} -->
The `hyperlinks_copy` function increments the reference count of a given `hyperlinks` structure and returns it.
- **Inputs**:
    - `hl`: A pointer to a `hyperlinks` structure whose reference count is to be incremented.
- **Control Flow**:
    - Increment the `references` field of the `hyperlinks` structure pointed to by `hl`.
    - Return the pointer `hl`.
- **Output**:
    - Returns the same `hyperlinks` structure pointer that was passed as input, with its reference count incremented.


---
### hyperlinks_reset<!-- {{#callable:hyperlinks_reset}} -->
The `hyperlinks_reset` function removes all hyperlinks from a given set without deallocating the set itself.
- **Inputs**:
    - `hl`: A pointer to a `struct hyperlinks` which represents the set of hyperlinks to be reset.
- **Control Flow**:
    - Iterate over each hyperlink in the `by_inner` tree of the given `hyperlinks` structure using `RB_FOREACH_SAFE`.
    - For each hyperlink, call [`hyperlinks_remove`](#hyperlinks_remove) to remove and deallocate it from the set.
- **Output**:
    - The function does not return any value; it performs its operation in-place on the provided `hyperlinks` structure.
- **Functions called**:
    - [`hyperlinks_remove`](#hyperlinks_remove)


---
### hyperlinks_free<!-- {{#callable:hyperlinks_free}} -->
The `hyperlinks_free` function decrements the reference count of a `hyperlinks` structure and frees its memory if the reference count reaches zero.
- **Inputs**:
    - `hl`: A pointer to a `hyperlinks` structure that is to be freed if its reference count is zero.
- **Control Flow**:
    - Decrement the `references` field of the `hyperlinks` structure pointed to by `hl`.
    - Check if the `references` field is now zero.
    - If the `references` field is zero, call [`hyperlinks_reset`](#hyperlinks_reset) to free all hyperlinks within the set.
    - Free the memory allocated for the `hyperlinks` structure itself.
- **Output**:
    - The function does not return any value; it performs memory management by potentially freeing the `hyperlinks` structure.
- **Functions called**:
    - [`hyperlinks_reset`](#hyperlinks_reset)


