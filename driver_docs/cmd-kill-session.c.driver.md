# Purpose
This C source code file is part of the tmux terminal multiplexer project and defines functionality for terminating a session within tmux. The primary function, [`cmd_kill_session_exec`](#cmd_kill_session_exec), is responsible for executing the command to destroy a session, which involves detaching all clients connected to it and removing any windows that are exclusively linked to the session. The code is structured to ensure that this operation is not performed accidentally, as indicated by the absence of an alias for the "kill-session" command. The command can be executed with specific options, such as `-a` to kill all sessions except the current one, or `-C` to clear alert flags from windows within the session.

The file defines a command entry structure, `cmd_kill_session_entry`, which specifies the command's name, arguments, usage, and the function to execute. This structure is part of a broader command handling system within tmux, where commands are registered and executed based on user input. The code interacts with various tmux components, such as sessions and windows, and utilizes data structures like red-black trees to manage these entities. The file does not define a public API but rather contributes to the internal command execution framework of tmux, providing a specific utility for session management.
# Imports and Dependencies

---
- `sys/types.h`
- `tmux.h`


# Global Variables

---
### cmd_kill_session_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_kill_session_entry` is a constant structure of type `cmd_entry` that defines the command 'kill-session' in the tmux application. It specifies the command's name, arguments, usage, target, flags, and the function to execute the command. This structure is used to manage the destruction of a session, detaching all clients and destroying windows linked only to that session.
- **Use**: This variable is used to define and execute the 'kill-session' command within the tmux application, allowing for session management and cleanup.


# Functions

---
### cmd_kill_session_exec<!-- {{#callable:cmd_kill_session_exec}} -->
The `cmd_kill_session_exec` function handles the execution of the 'kill-session' command, which can either clear alert flags from windows, destroy all sessions except the target, or destroy the target session itself.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item, which contains the target session information.
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args(self)` and the target session using `cmdq_get_target(item)`.
    - Check if the 'C' flag is present in the arguments using `args_has(args, 'C')`.
    - If 'C' flag is present, iterate over all window links in the target session, clear their alert flags, and redraw the session.
    - If 'C' flag is not present, check if the 'a' flag is present using `args_has(args, 'a')`.
    - If 'a' flag is present, iterate over all sessions and destroy each session except the target session.
    - If neither 'C' nor 'a' flags are present, destroy the target session.
    - Return `CMD_RETURN_NORMAL` to indicate normal command execution completion.
- **Output**:
    - The function returns an `enum cmd_retval` value, specifically `CMD_RETURN_NORMAL`, indicating successful execution of the command.


# Function Declarations (Public API)

---
### cmd_kill_session_exec<!-- {{#callable_declaration:cmd_kill_session_exec}} -->
Destroys a session, detaching all clients and destroying windows linked only to it.
- **Description**: This function is used to terminate a session in the tmux environment, which involves detaching all clients connected to the session and destroying any windows that are exclusively linked to it. It can be invoked with specific flags to alter its behavior: the '-C' flag clears alert flags from windows in the session, and the '-a' flag destroys all sessions except the target session. This function should be used with caution as it affects session state and client connections.
- **Inputs**:
    - `self`: A pointer to a 'struct cmd' representing the command context. Must not be null.
    - `item`: A pointer to a 'struct cmdq_item' representing the command queue item. Must not be null.
- **Output**: Returns 'CMD_RETURN_NORMAL' to indicate successful execution.
- **See also**: [`cmd_kill_session_exec`](#cmd_kill_session_exec)  (Implementation)


