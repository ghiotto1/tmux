# Purpose
This C source code file is part of a larger software system, likely related to the management of user access control within a server environment. The code defines and manages an Access Control List (ACL) for users, specifically focusing on their read and write permissions. The primary data structure used is a red-black tree, which efficiently organizes and manages user entries based on their user IDs (UIDs). The code provides functions to initialize the ACL, add or remove users, and modify their access permissions. It also includes functionality to display the current ACL state and to check and adjust client permissions based on the ACL.

The file is not a standalone executable but rather a component of a larger application, likely intended to be compiled and linked with other parts of the system. It does not define a public API or external interfaces but instead provides internal functions for managing user access within the server. The use of red-black trees for managing user entries suggests a focus on performance and scalability, as this data structure allows for efficient insertion, deletion, and lookup operations. The code also interacts with system-level functions to retrieve user information and manage client connections, indicating its integration with the operating system's user and process management facilities.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/stat.h`
- `sys/socket.h`
- `ctype.h`
- `pwd.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `tmux.h`


# Data Structures

---
### server_acl_user
- **Type**: `struct`
- **Members**:
    - `uid`: A unique identifier for the user, represented as a user ID (uid_t).
    - `flags`: An integer representing the access control flags for the user, such as read-only access.
    - `entry`: A Red-Black tree entry used to organize and manage the server_acl_user instances within a Red-Black tree structure.
- **Description**: The `server_acl_user` structure is designed to manage access control for users in a server environment. It contains a user ID (`uid`) to uniquely identify each user, a set of flags (`flags`) to specify access permissions (e.g., read-only access), and an `entry` field that integrates the structure into a Red-Black tree for efficient management and retrieval of user access control entries. This structure is part of a larger system that manages user permissions, allowing for operations such as adding, removing, and modifying user access rights.


# Functions

---
### server_acl_cmp<!-- {{#callable:server_acl_cmp}} -->
The `server_acl_cmp` function compares two `server_acl_user` structures based on their user IDs (UIDs).
- **Inputs**:
    - `user1`: A pointer to the first `server_acl_user` structure to be compared.
    - `user2`: A pointer to the second `server_acl_user` structure to be compared.
- **Control Flow**:
    - The function checks if the UID of `user1` is less than the UID of `user2`; if true, it returns -1.
    - If the first condition is false, it checks if the UID of `user1` is greater than the UID of `user2`; if true, it returns 1.
    - If neither condition is true, it implies the UIDs are equal, and the function returns 0.
- **Output**:
    - The function returns an integer: -1 if `user1`'s UID is less than `user2`'s UID, 1 if `user1`'s UID is greater than `user2`'s UID, and 0 if both UIDs are equal.


---
### server_acl_init<!-- {{#callable:server_acl_init}} -->
The `server_acl_init` function initializes the server access control list (ACL) by setting up the red-black tree and adding the root and current user to the ACL.
- **Inputs**:
    - None
- **Control Flow**:
    - Initialize the red-black tree `server_acl_entries` using `RB_INIT`.
    - Check if the current user ID (`getuid()`) is not root (0); if true, call `server_acl_user_allow(0)` to allow root access.
    - Call `server_acl_user_allow(getuid())` to allow the current user access.
- **Output**:
    - The function does not return any value; it initializes the server ACL state.
- **Functions called**:
    - [`server_acl_user_allow`](#server_acl_user_allow)


---
### server_acl_user_find<!-- {{#callable:server_acl_user_find}} -->
The function `server_acl_user_find` searches for a user entry in a red-black tree based on a given user ID (UID).
- **Inputs**:
    - `uid`: The user ID (UID) of the user to be searched for in the server access control list.
- **Control Flow**:
    - A `server_acl_user` structure named `find` is initialized with the provided UID.
    - The function `RB_FIND` is called with the red-black tree `server_acl_entries`, the address of the tree, and the address of the `find` structure to locate the user entry.
    - The result of `RB_FIND`, which is a pointer to the found `server_acl_user` structure or NULL if not found, is returned.
- **Output**:
    - A pointer to the `server_acl_user` structure if the user is found in the tree, or NULL if the user is not found.


---
### server_acl_display<!-- {{#callable:server_acl_display}} -->
The `server_acl_display` function iterates over a list of server access control list (ACL) entries and prints the username and access level (read-only or write) for each user.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which is used to print the output of the function.
- **Control Flow**:
    - Iterate over each `server_acl_user` entry in the `server_acl_entries` red-black tree using `RB_FOREACH`.
    - For each entry, skip the iteration if the user ID (`uid`) is 0, as this is typically reserved for the root user.
    - Retrieve the password entry for the user ID using `getpwuid`.
    - If the password entry is found, set the `name` to the user's name; otherwise, set it to "unknown".
    - Check the user's flags to determine if they have read-only access (`SERVER_ACL_READONLY`).
    - Print the user's name and access level ("(R)" for read-only or "(W)" for write) using `cmdq_print`.
- **Output**:
    - The function does not return a value; it outputs the list of users and their access levels to the provided `cmdq_item`.


---
### server_acl_user_allow<!-- {{#callable:server_acl_user_allow}} -->
The `server_acl_user_allow` function adds a user with a specified UID to the server access control list if they are not already present.
- **Inputs**:
    - `uid`: The user ID (UID) of the user to be allowed access.
- **Control Flow**:
    - Call [`server_acl_user_find`](#server_acl_user_find) to check if a user with the given UID already exists in the server ACL tree.
    - If the user does not exist (i.e., [`server_acl_user_find`](#server_acl_user_find) returns NULL), allocate memory for a new `server_acl_user` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc).
    - Set the UID of the newly allocated `server_acl_user` structure to the provided UID.
    - Insert the new user into the `server_acl_entries` red-black tree using `RB_INSERT`.
- **Output**:
    - The function does not return a value; it modifies the server ACL tree by adding a new user if necessary.
- **Functions called**:
    - [`server_acl_user_find`](#server_acl_user_find)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### server_acl_user_deny<!-- {{#callable:server_acl_user_deny}} -->
The `server_acl_user_deny` function removes a user from the server access control list (ACL) based on their user ID (UID).
- **Inputs**:
    - `uid`: The user ID (UID) of the user to be removed from the server ACL.
- **Control Flow**:
    - Call [`server_acl_user_find`](#server_acl_user_find) with the given UID to locate the user in the ACL tree.
    - Check if the user is found (i.e., the result is not NULL).
    - If the user is found, remove the user from the ACL tree using `RB_REMOVE`.
    - Free the memory allocated for the user structure.
- **Output**:
    - The function does not return any value; it performs the side effect of removing a user from the ACL tree and freeing associated memory.
- **Functions called**:
    - [`server_acl_user_find`](#server_acl_user_find)


---
### server_acl_user_allow_write<!-- {{#callable:server_acl_user_allow_write}} -->
The `server_acl_user_allow_write` function grants write access to a user identified by their UID by updating their access control flags and the flags of associated clients.
- **Inputs**:
    - `uid`: The user ID (UID) of the user for whom write access is to be granted.
- **Control Flow**:
    - The function begins by attempting to find the `server_acl_user` structure associated with the given UID using [`server_acl_user_find`](#server_acl_user_find).
    - If no user is found (i.e., the result is NULL), the function returns immediately, doing nothing further.
    - If a user is found, the function clears the `SERVER_ACL_READONLY` flag from the user's flags, thereby granting write access.
    - The function then iterates over all clients in the `clients` list using `TAILQ_FOREACH`.
    - For each client, it retrieves the client's UID using [`proc_get_peer_uid`](proc.c.driver.md#proc_get_peer_uid).
    - If the retrieved UID is valid and matches the user's UID, the function clears the `CLIENT_READONLY` flag from the client's flags, granting the client write access.
- **Output**:
    - The function does not return any value; it modifies the access control flags of the user and associated clients in place.
- **Functions called**:
    - [`server_acl_user_find`](#server_acl_user_find)
    - [`proc_get_peer_uid`](proc.c.driver.md#proc_get_peer_uid)


---
### server_acl_user_deny_write<!-- {{#callable:server_acl_user_deny_write}} -->
The function `server_acl_user_deny_write` sets a user and their associated clients to read-only mode by updating their flags.
- **Inputs**:
    - `uid`: The user ID (uid_t) of the user whose write access is to be denied.
- **Control Flow**:
    - Find the user in the server ACL entries using the provided UID.
    - If the user is not found, exit the function.
    - Set the user's flags to read-only by applying the SERVER_ACL_READONLY flag.
    - Iterate over all clients in the global clients list.
    - For each client, retrieve the UID of the peer associated with the client.
    - If the retrieved UID matches the user's UID, set the client's flags to read-only by applying the CLIENT_READONLY flag.
- **Output**:
    - The function does not return any value; it modifies the flags of the user and associated clients to indicate read-only access.
- **Functions called**:
    - [`server_acl_user_find`](#server_acl_user_find)
    - [`proc_get_peer_uid`](proc.c.driver.md#proc_get_peer_uid)


---
### server_acl_join<!-- {{#callable:server_acl_join}} -->
The `server_acl_join` function checks if a client's UID is in the server's access control list and sets the client to read-only if necessary, returning a success or failure status.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose access is being checked.
- **Control Flow**:
    - Retrieve the UID of the client using [`proc_get_peer_uid`](proc.c.driver.md#proc_get_peer_uid) with the client's peer information.
    - Check if the retrieved UID is invalid (equal to `(uid_t)-1`), and if so, return 0 indicating failure.
    - Search for the user in the server's ACL using [`server_acl_user_find`](#server_acl_user_find) with the retrieved UID.
    - If the user is not found in the ACL, return 0 indicating failure.
    - If the user is found and has the `SERVER_ACL_READONLY` flag set, update the client's flags to include `CLIENT_READONLY`.
    - Return 1 indicating success.
- **Output**:
    - The function returns an integer: 1 if the client's UID is found in the ACL and processed, or 0 if the UID is invalid or not found in the ACL.
- **Functions called**:
    - [`proc_get_peer_uid`](proc.c.driver.md#proc_get_peer_uid)
    - [`server_acl_user_find`](#server_acl_user_find)


---
### server_acl_get_uid<!-- {{#callable:server_acl_get_uid}} -->
The function `server_acl_get_uid` retrieves the user ID (UID) from a given `server_acl_user` structure.
- **Inputs**:
    - `user`: A pointer to a `server_acl_user` structure from which the UID is to be retrieved.
- **Control Flow**:
    - The function takes a single argument, a pointer to a `server_acl_user` structure.
    - It directly accesses the `uid` field of the provided `server_acl_user` structure.
    - The function returns the value of the `uid` field.
- **Output**:
    - The function returns the UID of the specified `server_acl_user` as a `uid_t` type.


