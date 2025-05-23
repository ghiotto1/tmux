# Purpose
This C source code file is part of a server application, specifically handling access control for users based on their user IDs (UIDs). It defines a set of functions and data structures to manage an Access Control List (ACL) for server users, allowing or denying them access and specifying their permissions (read-only or write access). The code utilizes a red-black tree data structure to efficiently store and manage user entries, with each entry represented by the `server_acl_user` structure. This structure includes a UID and a flags field to indicate the user's access level.

The file provides several key functions: [`server_acl_init`](#server_acl_init) initializes the ACL system, [`server_acl_user_allow`](#server_acl_user_allow) and [`server_acl_user_deny`](#server_acl_user_deny) manage user permissions by adding or removing users from the ACL, and [`server_acl_user_allow_write`](#server_acl_user_allow_write) and [`server_acl_user_deny_write`](#server_acl_user_deny_write) adjust a user's write permissions. Additionally, [`server_acl_display`](#server_acl_display) outputs the current ACL entries, and [`server_acl_join`](#server_acl_join) checks if a client is in the ACL and adjusts their permissions accordingly. This code is likely part of a larger application, such as a terminal multiplexer or a server management tool, where managing user access and permissions is crucial.
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
    - `entry`: A Red-Black tree entry used to organize and manage the server_acl_user instances within a Red-Black tree.
- **Description**: The `server_acl_user` structure is designed to manage access control for users in a server environment. It contains a user ID (`uid`) to uniquely identify each user, a set of flags (`flags`) to specify access permissions (e.g., read-only access), and an `entry` field for integrating the structure into a Red-Black tree. This data structure is part of an access control list (ACL) system that allows or denies user access to server resources, and it is used to efficiently manage and query user permissions.


# Functions

---
### server_acl_cmp<!-- {{#callable:server_acl_cmp}} -->
The `server_acl_cmp` function compares two `server_acl_user` structures based on their user IDs (UIDs) to determine their order.
- **Inputs**:
    - `user1`: A pointer to the first `server_acl_user` structure to be compared.
    - `user2`: A pointer to the second `server_acl_user` structure to be compared.
- **Control Flow**:
    - Check if the UID of `user1` is less than the UID of `user2`; if true, return -1 indicating `user1` should come before `user2`.
    - If the above condition is false, check if the UID of `user1` is greater than the UID of `user2`; if true, return 1 indicating `user1` should come after `user2`.
    - If neither condition is true, the UIDs are equal, and the function implicitly returns 0, indicating the two users are equivalent in terms of ordering.
- **Output**:
    - The function returns an integer: -1 if `user1` should come before `user2`, 1 if `user1` should come after `user2`, and 0 if they are equivalent.


---
### server_acl_init<!-- {{#callable:server_acl_init}} -->
The `server_acl_init` function initializes the server access control list (ACL) by setting up the ACL tree and allowing access to the root user and the current user.
- **Inputs**:
    - None
- **Control Flow**:
    - Initialize the red-black tree `server_acl_entries` using `RB_INIT`.
    - Check if the current user ID is not root (i.e., not 0) using `getuid() != 0`.
    - If the current user is not root, call `server_acl_user_allow(0)` to allow root user access.
    - Call `server_acl_user_allow(getuid())` to allow the current user access.
- **Output**:
    - The function does not return any value; it initializes the server ACL tree and sets up initial user access.
- **Functions called**:
    - [`server_acl_user_allow`](#server_acl_user_allow)


---
### server_acl_user_find<!-- {{#callable:server_acl_user_find}} -->
The `server_acl_user_find` function searches for a user entry in the server access control list (ACL) based on a given user ID (UID).
- **Inputs**:
    - `uid`: The user ID (UID) of the user to be searched in the server ACL.
- **Control Flow**:
    - A `server_acl_user` structure named `find` is initialized with the provided UID.
    - The function uses the `RB_FIND` macro to search for the `find` structure in the `server_acl_entries` red-black tree.
    - The search is based on the UID comparison defined by the `server_acl_cmp` function.
- **Output**:
    - The function returns a pointer to the `server_acl_user` structure if the user is found in the ACL; otherwise, it returns `NULL`.


---
### server_acl_display<!-- {{#callable:server_acl_display}} -->
The `server_acl_display` function iterates over a list of server access control list (ACL) entries and prints each user's name along with their access level (read-only or write).
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which is used to print the output of the function.
- **Control Flow**:
    - The function iterates over each entry in the `server_acl_entries` red-black tree using the `RB_FOREACH` macro.
    - For each entry, it checks if the user ID (`uid`) is 0 and skips it if true.
    - It retrieves the user's password entry using `getpwuid` to obtain the user's name; if the entry is not found, it defaults to "unknown".
    - It checks the user's flags to determine if the user has read-only access (`SERVER_ACL_READONLY`) or write access.
    - It prints the user's name and access level ("(R)" for read-only or "(W)" for write) using the `cmdq_print` function.
- **Output**:
    - The function does not return a value; it outputs the list of users and their access levels to the provided `cmdq_item`.


---
### server_acl_user_allow<!-- {{#callable:server_acl_user_allow}} -->
The `server_acl_user_allow` function adds a user with a specified UID to the server access control list if they are not already present.
- **Inputs**:
    - `uid`: The user ID (UID) of the user to be allowed access.
- **Control Flow**:
    - Call [`server_acl_user_find`](#server_acl_user_find) to check if a user with the given UID already exists in the server ACL tree.
    - If the user does not exist (i.e., [`server_acl_user_find`](#server_acl_user_find) returns NULL), allocate memory for a new `server_acl_user` structure using `xcalloc`.
    - Set the UID of the newly allocated `server_acl_user` structure to the provided UID.
    - Insert the new user into the `server_acl_entries` red-black tree using `RB_INSERT`.
- **Output**:
    - The function does not return a value; it modifies the server ACL tree by adding a new user if necessary.
- **Functions called**:
    - [`server_acl_user_find`](#server_acl_user_find)


---
### server_acl_user_deny<!-- {{#callable:server_acl_user_deny}} -->
The `server_acl_user_deny` function removes a user from the server access control list based on their user ID.
- **Inputs**:
    - `uid`: The user ID (uid_t) of the user to be removed from the server access control list.
- **Control Flow**:
    - Call [`server_acl_user_find`](#server_acl_user_find) to locate the user entry in the server access control list using the provided UID.
    - Check if the user entry is not NULL, indicating the user exists in the list.
    - If the user exists, remove the user from the `server_acl_entries` red-black tree using `RB_REMOVE`.
    - Free the memory allocated for the user entry.
- **Output**:
    - The function does not return any value; it performs the action of removing a user from the access control list if they exist.
- **Functions called**:
    - [`server_acl_user_find`](#server_acl_user_find)


---
### server_acl_user_allow_write<!-- {{#callable:server_acl_user_allow_write}} -->
The `server_acl_user_allow_write` function grants write access to a user by updating their access control list (ACL) entry and modifying client flags accordingly.
- **Inputs**:
    - `uid`: The user ID (uid_t) of the user for whom write access is to be granted.
- **Control Flow**:
    - Find the user in the server ACL entries using the provided UID.
    - If the user is not found, exit the function.
    - Clear the SERVER_ACL_READONLY flag for the user to allow write access.
    - Iterate over all clients in the clients list.
    - For each client, retrieve the UID of the peer associated with the client.
    - If the retrieved UID matches the user's UID, clear the CLIENT_READONLY flag for that client to allow write access.
- **Output**:
    - The function does not return any value; it modifies the ACL and client flags to grant write access.
- **Functions called**:
    - [`server_acl_user_find`](#server_acl_user_find)


---
### server_acl_user_deny_write<!-- {{#callable:server_acl_user_deny_write}} -->
The `server_acl_user_deny_write` function sets a user and their associated clients to read-only mode by updating their flags.
- **Inputs**:
    - `uid`: The user ID (uid_t) of the user whose write access is to be denied.
- **Control Flow**:
    - Find the user in the server ACL entries using the provided UID.
    - If the user is not found, exit the function.
    - Set the user's flags to read-only by applying the SERVER_ACL_READONLY bitmask.
    - Iterate over all clients in the global clients list.
    - For each client, retrieve the UID of the peer associated with the client.
    - If the retrieved UID is valid and matches the user's UID, set the client's flags to read-only by applying the CLIENT_READONLY bitmask.
- **Output**:
    - The function does not return any value; it modifies the flags of the user and associated clients to indicate read-only access.
- **Functions called**:
    - [`server_acl_user_find`](#server_acl_user_find)


---
### server_acl_join<!-- {{#callable:server_acl_join}} -->
The `server_acl_join` function checks if a client's UID is in the server's access control list and sets the client to read-only if necessary, returning a success or failure status.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client whose access is being checked.
- **Control Flow**:
    - Retrieve the UID of the client using `proc_get_peer_uid` with the client's peer information.
    - If the UID retrieval fails (returns -1), return 0 indicating failure.
    - Search for the user in the server's ACL using [`server_acl_user_find`](#server_acl_user_find) with the retrieved UID.
    - If the user is not found in the ACL, return 0 indicating failure.
    - Check if the user has the `SERVER_ACL_READONLY` flag set; if so, set the client's `CLIENT_READONLY` flag.
    - Return 1 indicating success.
- **Output**:
    - The function returns an integer: 1 if the client's UID is found in the ACL and processed, or 0 if the UID is not found or an error occurs.
- **Functions called**:
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


