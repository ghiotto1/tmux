# Purpose
This C source code file is part of the tmux terminal multiplexer project, specifically implementing the functionality for creating and managing new sessions. The file defines two command entries: `new-session` and `has-session`, which are integral to session management within tmux. The `new-session` command is responsible for creating a new tmux session and optionally attaching it to the current terminal, unless the `-d` flag is specified to detach it. The `has-session` command checks for the existence of a session, returning success if the session is found. The core functionality is encapsulated in the [`cmd_new_session_exec`](#cmd_new_session_exec) function, which handles the creation of a new session, setting up its environment, and managing session groups if specified.

The file includes several technical components such as session name validation, environment setup, terminal size configuration, and session group management. It uses various system and library calls to interact with the terminal and manage session attributes. The code is structured to handle different command-line arguments, allowing for flexible session creation and management. This file is a crucial part of the tmux command execution framework, providing a public API for session-related operations and integrating with the broader tmux server-client architecture.
# Imports and Dependencies

---
- `sys/types.h`
- `errno.h`
- `fcntl.h`
- `stdlib.h`
- `string.h`
- `termios.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### cmd_new_session_entry
- **Type**: ``const struct cmd_entry``
- **Description**: The `cmd_new_session_entry` is a constant structure of type `cmd_entry` that defines the properties and behavior of the 'new-session' command in the tmux application. It includes fields such as the command name, alias, argument specifications, usage instructions, target session details, command flags, and the function to execute the command.
- **Use**: This variable is used to register and define the behavior of the 'new-session' command within the tmux command processing system.


---
### cmd_has_session_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_has_session_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'has-session' command in the tmux application. It includes fields such as the command name, alias, arguments, usage, target, flags, and the execution function associated with the command.
- **Use**: This variable is used to define and register the 'has-session' command, allowing tmux to recognize and execute this command when invoked by the user.


# Functions

---
### cmd_new_session_exec<!-- {{#callable:cmd_new_session_exec}} -->
The `cmd_new_session_exec` function creates a new session in tmux, optionally attaching it to a client, and handles various configurations and error conditions.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve command arguments and current/target states from the command queue item.
    - Check if the command is a 'has-session' entry and return success if so.
    - Validate session name and check for duplicate sessions if a new session name is provided.
    - Determine if the session should be part of a session group and handle group-related logic.
    - Set the detached flag based on the presence of a client and client control flags.
    - Determine the working directory for the new session, defaulting to the client's current directory if not specified.
    - Check for nested sessions and save terminal settings if the client is not detached or already attached.
    - Open the terminal for the client if necessary and handle errors.
    - Determine the default and new session sizes based on arguments and client settings.
    - Create a new session with the specified options, environment, and terminal settings.
    - Spawn the initial window in the new session and handle errors if window creation fails.
    - Add the session to a session group if specified and synchronize the group.
    - Attach the client to the new session if not detached, setting flags and sending readiness messages as needed.
    - Print session information if requested by the user.
    - Set the client as attached and update session state if not detached.
    - Insert hooks and show configuration causes if applicable.
    - Free allocated resources and return success or error based on the execution outcome.
- **Output**:
    - Returns `CMD_RETURN_NORMAL` on successful session creation and attachment, or `CMD_RETURN_ERROR` if an error occurs during the process.


# Function Declarations (Public API)

---
### cmd_new_session_exec<!-- {{#callable_declaration:cmd_new_session_exec}} -->
Create a new session and optionally attach it to a terminal.
- **Description**: This function is used to create a new session in the tmux environment, with the option to attach it to the current terminal unless the detached mode is specified. It can be used to start a new session with specific parameters such as session name, window name, and initial command. The function handles various options for session creation, including setting the session size, environment variables, and session group management. It must be called with a valid command structure and command queue item, and it will return an appropriate status based on the success or failure of the session creation.
- **Inputs**:
    - `self`: A pointer to a 'cmd' structure representing the command to be executed. Must not be null.
    - `item`: A pointer to a 'cmdq_item' structure representing the command queue item. Must not be null.
- **Output**: Returns an enum 'cmd_retval' indicating the success (CMD_RETURN_NORMAL) or failure (CMD_RETURN_ERROR) of the session creation.
- **See also**: [`cmd_new_session_exec`](#cmd_new_session_exec)  (Implementation)


