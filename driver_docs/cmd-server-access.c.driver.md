# Purpose
This C source code file is part of the tmux project, specifically implementing functionality to control access to a tmux server session. The file defines a command, "server-access," which allows for managing user permissions related to the tmux server. The command can be used to add or deny access to users, as well as to allow or deny write permissions. The code includes functions to execute these operations, such as [`cmd_server_access_exec`](#cmd_server_access_exec) for executing the command and [`cmd_server_access_deny`](#cmd_server_access_deny) for denying access to a user. The command is defined with various flags (`-a`, `-d`, `-l`, `-r`, `-w`) that specify the action to be taken, such as adding or denying access, listing current access control lists, or modifying write permissions.

The file is structured to integrate with the broader tmux command framework, utilizing structures and functions like `cmd_entry`, `cmdq_item`, and `args` to handle command execution and argument parsing. It interacts with user account information through the `passwd` structure and system calls like `getpwnam` to retrieve user details. The code ensures that certain operations are restricted, such as preventing changes to the server owner's access and disallowing conflicting flags. This file is a focused component of the tmux codebase, providing a specific utility for session access control, and is intended to be compiled and linked within the larger tmux application.
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
- **Description**: The `cmd_server_access_entry` is a constant structure of type `cmd_entry` that defines a command named 'server-access'. It specifies the command's name, arguments, usage, flags, and the function to execute when the command is invoked. This structure is used to control access to a session by allowing or denying user permissions based on the provided arguments.
- **Use**: This variable is used to define the 'server-access' command, detailing its behavior and execution within the application.


# Functions

---
### cmd_server_access_deny<!-- {{#callable:cmd_server_access_deny}} -->
The function `cmd_server_access_deny` denies access to a server for a specified user by setting an exit message and flagging the client's connection for termination.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which represents a command queue item used for error reporting.
    - `pw`: A pointer to a `passwd` structure, which contains user information, including the user ID (`pw_uid`) and user name (`pw_name`).
- **Control Flow**:
    - The function attempts to find the server access control list (ACL) entry for the user using `server_acl_user_find` with the user's UID (`pw->pw_uid`).
    - If the user is not found in the ACL, an error message is reported using `cmdq_error`, and the function returns `CMD_RETURN_ERROR`.
    - If the user is found, the function iterates over all connected clients using `TAILQ_FOREACH` on the `clients` list.
    - For each client, it retrieves the client's UID using `proc_get_peer_uid` and compares it with the UID from the ACL entry (`server_acl_get_uid`).
    - If the UIDs match, the client's `exit_message` is set to "access not allowed", and the `CLIENT_EXIT` flag is set to indicate that the client should be disconnected.
    - Finally, the function calls `server_acl_user_deny` to update the ACL to deny access for the user, and returns `CMD_RETURN_NORMAL`.
- **Output**:
    - The function returns an `enum cmd_retval`, which is either `CMD_RETURN_ERROR` if the user is not found in the ACL, or `CMD_RETURN_NORMAL` if the operation completes successfully.


---
### cmd_server_access_exec<!-- {{#callable:cmd_server_access_exec}} -->
The `cmd_server_access_exec` function manages user access control to a server session by processing command-line arguments to add, remove, or modify user permissions.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item being processed.
- **Control Flow**:
    - Retrieve command arguments using `cmd_get_args` and target client using `cmdq_get_target_client`.
    - If the '-l' flag is present, display the server access control list and return normal status.
    - If no user argument is provided, log an error and return an error status.
    - Format the user name from arguments and retrieve the corresponding password entry using `getpwnam`.
    - If the user is unknown, log an error and return an error status.
    - Check if the user is the server owner or the current user, log an error if true, and return an error status.
    - Ensure mutually exclusive flags '-a' and '-d', and '-w' and '-r' are not used together, logging errors if they are.
    - If '-d' is specified, call [`cmd_server_access_deny`](#cmd_server_access_deny) to deny access and return its result.
    - If '-a' is specified, check if the user is already added, log an error if true, otherwise allow the user.
    - If '-r' or '-w' is specified, imply '-a' if the user does not exist, and allow the user.
    - If '-w' is specified, check if the user exists, log an error if not, allow write access, and return normal status.
    - If '-r' is specified, check if the user exists, log an error if not, deny write access, and return normal status.
    - Return normal status if no errors occur.
- **Output**:
    - The function returns an `enum cmd_retval` indicating the success or failure of the command execution, with possible values being `CMD_RETURN_NORMAL` or `CMD_RETURN_ERROR`.
- **Functions called**:
    - [`cmd_server_access_deny`](#cmd_server_access_deny)


# Function Declarations (Public API)

---
### cmd_server_access_exec<!-- {{#callable_declaration:cmd_server_access_exec}} -->
Controls access to a server session based on user permissions.
- **Description**: This function manages access control for a server session by modifying user permissions according to specified command-line arguments. It can display current access lists, add or remove users, and adjust read/write permissions. The function requires a user argument unless displaying the access list. It ensures that conflicting options are not used together and checks for valid user entries. It should be used when there is a need to modify user access to a server session, ensuring that the user does not own the server and that valid options are provided.
- **Inputs**:
    - `self`: A pointer to the command structure representing the current command. Must not be null.
    - `item`: A pointer to the command queue item, representing the context in which the command is executed. Must not be null.
- **Output**: Returns CMD_RETURN_NORMAL on successful execution or CMD_RETURN_ERROR if an error occurs, such as invalid arguments or user not found.
- **See also**: [`cmd_server_access_exec`](#cmd_server_access_exec)  (Implementation)


