# Purpose
This C source code file is part of the tmux terminal multiplexer project and provides functionality for detaching and suspending clients. The file defines two command entries: `detach-client` and `suspend-client`, each with specific options and usage patterns. The `detach-client` command allows for detaching a client from a session, with options to specify a target session, execute a shell command upon detachment, or detach all clients except the current one. The `suspend-client` command, on the other hand, is used to suspend a client, effectively pausing its activity. Both commands utilize the [`cmd_detach_client_exec`](#cmd_detach_client_exec) function to execute their respective operations, which involves interacting with the server to either detach or suspend clients based on the provided arguments.

The file is structured to integrate with the broader tmux system, using internal APIs and data structures such as `cmd`, `cmdq_item`, `client`, and `session`. It defines public command entries that are likely registered with the tmux command handling system, allowing users to invoke these commands through the tmux interface. The code is designed to be modular, with clear separation between command definition and execution logic, and it leverages the existing tmux infrastructure to manage client sessions effectively. The use of flags and argument parsing indicates a robust design that accommodates various user scenarios, enhancing the flexibility and usability of the tmux client management features.
# Imports and Dependencies

---
- `sys/types.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_detach_client_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_detach_client_entry` is a constant structure of type `cmd_entry` that defines the command for detaching a client in the tmux application. It includes the command name, alias, arguments, usage information, source, flags, and the execution function associated with the command. This structure is used to register and handle the 'detach-client' command within the tmux command framework.
- **Use**: This variable is used to define and register the 'detach-client' command, allowing users to detach a client from a session in tmux.


---
### cmd_suspend_client_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_suspend_client_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'suspend-client' command in the application. It includes fields such as the command name, alias, arguments, usage, flags, and the function to execute when the command is invoked.
- **Use**: This variable is used to register and define the behavior of the 'suspend-client' command within the application, allowing clients to be suspended.


# Functions

---
### cmd_detach_client_exec<!-- {{#callable:cmd_detach_client_exec}} -->
The `cmd_detach_client_exec` function detaches a client from a session or executes a command on the client based on the provided arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args` and the source state using `cmdq_get_source`.
    - Get the target client using `cmdq_get_target_client`.
    - Check if the command entry is `cmd_suspend_client_entry`; if so, suspend the client and return `CMD_RETURN_NORMAL`.
    - Determine the message type based on the presence of the 'P' argument, setting it to `MSG_DETACHKILL` or `MSG_DETACH`.
    - If the 's' argument is present, iterate over all clients and detach or execute a command on those in the specified session, then return `CMD_RETURN_STOP`.
    - If the 'a' argument is present, iterate over all clients and detach or execute a command on all except the target client, then return `CMD_RETURN_NORMAL`.
    - If a command is specified, execute it on the target client; otherwise, detach the target client, then return `CMD_RETURN_STOP`.
- **Output**:
    - The function returns an `enum cmd_retval` indicating the result of the command execution, either `CMD_RETURN_NORMAL` or `CMD_RETURN_STOP`.


# Function Declarations (Public API)

---
### cmd_detach_client_exec<!-- {{#callable_declaration:cmd_detach_client_exec}} -->
Detach a client from a session.
- **Description**: This function is used to detach a client from a session in a tmux environment. It can be used to detach a specific client, all clients except the target, or all clients attached to a specific session. The function can also execute a shell command upon detachment if specified. It must be called with a valid command structure and command queue item. The behavior of the detachment can be modified using various flags, such as executing a command or detaching all clients.
- **Inputs**:
    - `self`: A pointer to the command structure representing the detach-client command. Must not be null.
    - `item`: A pointer to the command queue item that provides context for the command execution, including the target client and source session. Must not be null.
- **Output**: Returns an enum value indicating the result of the command execution, which can be CMD_RETURN_NORMAL or CMD_RETURN_STOP.
- **See also**: [`cmd_detach_client_exec`](#cmd_detach_client_exec)  (Implementation)


