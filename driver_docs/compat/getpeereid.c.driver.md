# Purpose
This C source code file provides a function named [`getpeereid`](#getpeereid), which is designed to retrieve the effective user ID (UID) and group ID (GID) of the peer connected to a socket. The function is implemented to support different system environments by using conditional compilation directives. It checks for the availability of specific system features or headers, such as `SO_PEERCRED` and `getpeerucred`, to determine the appropriate method for obtaining the peer credentials. If these features are not available, it defaults to using the effective UID and GID of the current process. This approach ensures compatibility across various Unix-like operating systems.

The file includes necessary system headers like `<sys/types.h>`, `<sys/socket.h>`, and `<unistd.h>`, which are essential for socket operations and user/group ID functions. The presence of conditional inclusion of `<ucred.h>` and the use of a custom header `"compat.h"` suggest that the code is designed to be portable and adaptable to different system environments. The function [`getpeereid`](#getpeereid) is likely intended to be part of a larger codebase, possibly a library or a utility that requires peer credential verification for security or logging purposes. The code does not define a public API or external interface directly but provides a crucial utility function that can be used internally or exposed through a higher-level interface.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/socket.h`
- `stdio.h`
- `unistd.h`
- `ucred.h`
- `compat.h`


# Functions

---
### getpeereid<!-- {{#callable:getpeereid}} -->
The `getpeereid` function retrieves the effective user ID (UID) and group ID (GID) of the peer connected to a socket, using different methods depending on the system's capabilities.
- **Inputs**:
    - `s`: The socket file descriptor for which the peer's UID and GID are to be retrieved.
    - `uid`: A pointer to a `uid_t` variable where the peer's user ID will be stored.
    - `gid`: A pointer to a `gid_t` variable where the peer's group ID will be stored.
- **Control Flow**:
    - Check if the `HAVE_SO_PEERCRED` macro is defined, indicating support for the `SO_PEERCRED` socket option.
    - If `HAVE_SO_PEERCRED` is defined, use `getsockopt` to retrieve the peer credentials and store the UID and GID in the provided pointers; return -1 on failure, 0 on success.
    - If `HAVE_GETPEERUCRED` is defined instead, use `getpeerucred` to obtain the peer credentials, extract the UID and GID using `ucred_geteuid` and `ucred_getrgid`, and free the credentials object; return -1 on failure, 0 on success.
    - If neither macro is defined, fall back to using `geteuid` and `getegid` to set the UID and GID to the effective user and group IDs of the current process, and return 0.
- **Output**:
    - The function returns 0 on success, indicating that the UID and GID have been successfully retrieved and stored in the provided pointers, or -1 on failure.


