# Purpose
This C source code file is part of the tmux terminal multiplexer project and provides functionality for detaching and suspending clients from a tmux session. The file defines two command entries, `detach-client` and `suspend-client`, each with specific options and usage instructions. The `detach-client` command allows for detaching a client from a session, with options to specify a target session, execute a shell command upon detachment, or detach all clients except the current one. The `suspend-client` command, on the other hand, is used to suspend a client, effectively pausing its activity.

The core functionality is implemented in the [`cmd_detach_client_exec`](#cmd_detach_client_exec) function, which handles the logic for both detaching and suspending clients based on the command invoked and the arguments provided. The function interacts with the tmux server to either execute a command on the client or detach it, depending on the specified options. The file is structured to be part of a larger codebase, likely imported and used by other components of tmux, and it does not define a standalone executable. Instead, it provides specific command implementations that integrate with the tmux command queue and client management system.
# Imports and Dependencies

---
- `sys/types.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_detach_client_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_detach_client_entry` is a constant structure of type `cmd_entry` that defines the command entry for detaching a client in the tmux application. It includes the command name, alias, arguments, usage information, source, flags, and the execution function associated with the command. This structure is used to register and handle the 'detach-client' command within the tmux command framework.
- **Use**: This variable is used to define and register the 'detach-client' command, allowing it to be executed with specified options and behavior in the tmux application.


---
### cmd_suspend_client_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_suspend_client_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'suspend-client' command in the application. It includes the command's name, alias, argument specifications, usage information, flags, and the function to execute when the command is invoked.
- **Use**: This variable is used to register and define the behavior of the 'suspend-client' command within the application, allowing it to be recognized and executed appropriately.


# Functions

---
### cmd_detach_client_exec<!-- {{#callable:cmd_detach_client_exec}} -->
The `cmd_detach_client_exec` function detaches a client or executes a command on a client based on the provided arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the arguments associated with the command using [`cmd_get_args`](cmd.c.driver.md#cmd_get_args).
    - Get the source state and target client from the command queue item using [`cmdq_get_source`](cmd-queue.c.driver.md#cmdq_get_source) and [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client).
    - Check if the command entry is `cmd_suspend_client_entry`; if so, suspend the target client and return `CMD_RETURN_NORMAL`.
    - Determine the message type (`MSG_DETACHKILL` or `MSG_DETACH`) based on the presence of the 'P' argument.
    - If the 's' argument is present, iterate over all clients and detach or execute a command on those associated with the specified session, then return `CMD_RETURN_STOP`.
    - If the 'a' argument is present, iterate over all clients and detach or execute a command on those not equal to the target client, then return `CMD_RETURN_NORMAL`.
    - If a command is specified with the 'E' argument, execute it on the target client; otherwise, detach the target client with the determined message type.
    - Return `CMD_RETURN_STOP` after processing the target client.
- **Output**:
    - The function returns an `enum cmd_retval` indicating the result of the command execution, either `CMD_RETURN_NORMAL` or `CMD_RETURN_STOP`.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_source`](cmd-queue.c.driver.md#cmdq_get_source)
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`cmd_get_entry`](cmd.c.driver.md#cmd_get_entry)
    - [`server_client_suspend`](server-client.c.driver.md#server_client_suspend)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`server_client_exec`](server-client.c.driver.md#server_client_exec)
    - [`server_client_detach`](server-client.c.driver.md#server_client_detach)


