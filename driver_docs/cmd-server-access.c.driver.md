# Purpose
This C source code file is part of the tmux project, specifically implementing functionality to control access to a tmux server session. The file defines a command, "server-access," which allows for managing user permissions related to the tmux server. The command can be used to add or deny access to users, as well as to allow or deny write permissions. The code includes functions to execute these operations, such as [`cmd_server_access_exec`](#cmd_server_access_exec) for executing the command and [`cmd_server_access_deny`](#cmd_server_access_deny) for denying access to a user. The command is defined with various flags (`-a`, `-d`, `-l`, `-r`, `-w`) that specify the action to be taken, such as adding, denying, listing, allowing read, or allowing write access.

The file is structured to integrate with the broader tmux command framework, utilizing structures like `cmd_entry` to define command properties and behaviors. It interacts with user account information through the `passwd` structure and system calls to manage access control lists (ACLs) for the tmux server. The code ensures that certain constraints are respected, such as preventing changes to the server owner's access and disallowing conflicting flags. This file is a focused component of the tmux codebase, providing a specific utility for session access management, and is intended to be compiled and linked within the larger tmux application.
# Imports and Dependencies

---
- `sys/stat.h`
- `sys/types.h`
- `pwd.h`
- `stdio.h`
- `string.h`
- `stdlib.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### cmd_server_access_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_server_access_entry` is a constant structure of type `cmd_entry` that defines a command named 'server-access'. It specifies the command's name, arguments, usage, flags, and the function to execute when the command is invoked. This structure is used to manage access control to a server session, allowing for operations such as adding, denying, or modifying user access.
- **Use**: This variable is used to register and define the behavior of the 'server-access' command within the application, enabling control over user access to server sessions.


# Functions

---
### cmd_server_access_deny<!-- {{#callable:cmd_server_access_deny}} -->
The `cmd_server_access_deny` function denies access to a server for a specified user by setting an exit message and flagging the client's connection for termination.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which represents a command queue item used for error reporting.
    - `pw`: A pointer to a `passwd` structure, which contains user information, including the user ID (`pw_uid`) and user name (`pw_name`).
- **Control Flow**:
    - The function attempts to find the server ACL user associated with the provided user ID (`pw->pw_uid`).
    - If the user is not found, it logs an error message using `cmdq_error` and returns `CMD_RETURN_ERROR`.
    - If the user is found, it iterates over all connected clients using `TAILQ_FOREACH`.
    - For each client, it retrieves the client's user ID using [`proc_get_peer_uid`](proc.c.driver.md#proc_get_peer_uid).
    - If the client's user ID matches the server ACL user's ID, it sets the client's `exit_message` to "access not allowed" and flags the client for exit by setting `CLIENT_EXIT`.
    - Finally, it calls [`server_acl_user_deny`](server-acl.c.driver.md#server_acl_user_deny) to deny access for the user ID and returns `CMD_RETURN_NORMAL`.
- **Output**:
    - The function returns an `enum cmd_retval`, which is either `CMD_RETURN_ERROR` if the user is not found or `CMD_RETURN_NORMAL` if the operation completes successfully.
- **Functions called**:
    - [`server_acl_user_find`](server-acl.c.driver.md#server_acl_user_find)
    - [`proc_get_peer_uid`](proc.c.driver.md#proc_get_peer_uid)
    - [`server_acl_get_uid`](server-acl.c.driver.md#server_acl_get_uid)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`server_acl_user_deny`](server-acl.c.driver.md#server_acl_user_deny)


---
### cmd_server_access_exec<!-- {{#callable:cmd_server_access_exec}} -->
The `cmd_server_access_exec` function manages server access control by processing command arguments to display, add, deny, or modify user access permissions.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item, which provides context for the command execution.
- **Control Flow**:
    - Retrieve the command arguments using [`cmd_get_args`](cmd.c.driver.md#cmd_get_args) and the target client using [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client).
    - Check if the '-l' flag is present; if so, display the server ACL and return normal.
    - If no user argument is provided, log an error and return an error status.
    - Format the user name from the arguments and attempt to retrieve the corresponding password entry using `getpwnam`.
    - If the user is not found, log an error and return an error status.
    - Check if the user is the server owner or the current user; if so, log an error and return an error status.
    - Ensure that '-a' and '-d' flags, as well as '-w' and '-r' flags, are not used together; if they are, log an error and return an error status.
    - If the '-d' flag is present, call [`cmd_server_access_deny`](#cmd_server_access_deny) to deny access and return its result.
    - If the '-a' flag is present, check if the user is already added; if not, allow the user and proceed to check '-r' or '-w'.
    - If '-r' or '-w' is present without '-a', imply '-a' and allow the user if not already present.
    - If the '-w' flag is present, check if the user exists; if not, log an error and return an error status. Otherwise, allow write access and return normal.
    - If the '-r' flag is present, check if the user exists; if not, log an error and return an error status. Otherwise, deny write access and return normal.
    - Return normal if no errors occur.
- **Output**:
    - The function returns an `enum cmd_retval` indicating the success or failure of the command execution, with possible values being `CMD_RETURN_NORMAL` or `CMD_RETURN_ERROR`.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`server_acl_display`](server-acl.c.driver.md#server_acl_display)
    - [`args_count`](arguments.c.driver.md#args_count)
    - [`format_single`](format.c.driver.md#format_single)
    - [`args_string`](arguments.c.driver.md#args_string)
    - [`cmd_server_access_deny`](#cmd_server_access_deny)
    - [`server_acl_user_find`](server-acl.c.driver.md#server_acl_user_find)
    - [`server_acl_user_allow`](server-acl.c.driver.md#server_acl_user_allow)
    - [`server_acl_user_allow_write`](server-acl.c.driver.md#server_acl_user_allow_write)
    - [`server_acl_user_deny_write`](server-acl.c.driver.md#server_acl_user_deny_write)


