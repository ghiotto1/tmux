# Purpose
This C source code file is part of the `tmux` project, a terminal multiplexer that allows users to manage multiple terminal sessions from a single window. The file defines functionality for creating and managing new sessions within `tmux`. It includes the implementation of the `new-session` command, which is responsible for creating a new session and optionally attaching it to the current terminal. The command can be customized with various options, such as specifying a session name, starting directory, environment variables, and window dimensions. The file also defines the `has-session` command, which checks for the existence of a session.

The core function, [`cmd_new_session_exec`](#cmd_new_session_exec), handles the execution logic for creating a new session. It processes command-line arguments, manages session naming and grouping, and handles terminal settings. The function also manages client attachment and session synchronization, ensuring that new sessions are properly initialized and integrated into the existing `tmux` environment. The file is structured to be part of a larger codebase, with dependencies on other components like session management, client handling, and environment configuration. It does not define a standalone executable but rather contributes to the broader functionality of the `tmux` application by providing a specific command implementation.
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
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_new_session_entry` is a constant structure of type `cmd_entry` that defines the properties and behavior of the 'new-session' command in the tmux application. It includes fields such as the command name, alias, argument specifications, usage instructions, target session details, command flags, and the execution function pointer. This structure is used to encapsulate all necessary information for executing the 'new-session' command, which is responsible for creating a new session in tmux.
- **Use**: This variable is used to define and execute the 'new-session' command within the tmux application, allowing users to create new sessions with specified parameters.


---
### cmd_has_session_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_has_session_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'has-session' command in the tmux application. It specifies the command's name, alias, arguments, usage, target, flags, and execution function. This structure is used to register and handle the 'has-session' command within the tmux command processing framework.
- **Use**: This variable is used to define and register the 'has-session' command in tmux, allowing it to be executed and processed by the command handling system.


# Functions

---
### cmd_new_session_exec<!-- {{#callable:cmd_new_session_exec}} -->
The `cmd_new_session_exec` function creates a new session in tmux, optionally attaching it to a client, and handles various configurations and error conditions.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve command arguments and current/target states from the command queue item.
    - Check if the command is a 'has-session' command and return success if so.
    - Validate arguments, ensuring no command or window name is given with a target session.
    - Retrieve and validate the session name if provided, checking for duplicates.
    - If the '-A' flag is set, attempt to attach to an existing session or create a new one if it doesn't exist.
    - Determine if the session should be part of a session group and handle accordingly.
    - Set the detached flag based on the presence of a client and its control status.
    - Check if the client is already attached to a session.
    - Determine the working directory for the new session, using the client's current directory if not specified.
    - If the client is new, check for nested sessions and save terminal settings if necessary.
    - Open the terminal for the client if not detached and not already attached.
    - Determine the default session size based on arguments or client settings.
    - Create a new session with the specified options, environment, and terminal settings.
    - Spawn the initial window in the new session, handling errors if window creation fails.
    - If part of a session group, add the session to the group and synchronize.
    - Attach the client to the new session if not detached, setting flags and sending readiness messages as needed.
    - Print session information if requested by the user.
    - Mark the client as attached if not detached and update session state.
    - Insert hooks and show configuration causes if applicable.
    - Free allocated resources and return success or error based on the execution outcome.
- **Output**:
    - Returns `CMD_RETURN_NORMAL` on successful session creation and attachment, or `CMD_RETURN_ERROR` if an error occurs during the process.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_current`](cmd-queue.c.driver.md#cmdq_get_current)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`args_count`](arguments.c.driver.md#args_count)
    - [`cmd_get_entry`](cmd.c.driver.md#cmd_get_entry)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`format_single`](format.c.driver.md#format_single)
    - [`session_check_name`](session.c.driver.md#session_check_name)
    - [`session_find`](session.c.driver.md#session_find)
    - [`cmd_attach_session`](cmd-attach-session.c.driver.md#cmd_attach_session)
    - [`session_group_find`](session.c.driver.md#session_group_find)
    - [`session_group_contains`](session.c.driver.md#session_group_contains)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`server_client_get_cwd`](server-client.c.driver.md#server_client_get_cwd)
    - [`server_client_check_nested`](server-client.c.driver.md#server_client_check_nested)
    - [`server_client_open`](server-client.c.driver.md#server_client_open)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`options_create`](options.c.driver.md#options_create)
    - [`environ_create`](environ.c.driver.md#environ_create)
    - [`environ_update`](environ.c.driver.md#environ_update)
    - [`args_first_value`](arguments.c.driver.md#args_first_value)
    - [`environ_put`](environ.c.driver.md#environ_put)
    - [`args_next_value`](arguments.c.driver.md#args_next_value)
    - [`session_create`](session.c.driver.md#session_create)
    - [`args_to_vector`](arguments.c.driver.md#args_to_vector)
    - [`spawn_window`](spawn.c.driver.md#spawn_window)
    - [`session_destroy`](session.c.driver.md#session_destroy)
    - [`session_group_new`](session.c.driver.md#session_group_new)
    - [`session_group_add`](session.c.driver.md#session_group_add)
    - [`session_group_synchronize_to`](session.c.driver.md#session_group_synchronize_to)
    - [`session_select`](session.c.driver.md#session_select)
    - [`notify_session`](notify.c.driver.md#notify_session)
    - [`server_client_set_flags`](server-client.c.driver.md#server_client_set_flags)
    - [`proc_send`](proc.c.driver.md#proc_send)
    - [`server_client_set_session`](server-client.c.driver.md#server_client_set_session)
    - [`cmdq_get_flags`](cmd-queue.c.driver.md#cmdq_get_flags)
    - [`server_client_set_key_table`](server-client.c.driver.md#server_client_set_key_table)
    - [`cmd_find_from_session`](cmd-find.c.driver.md#cmd_find_from_session)
    - [`cfg_show_causes`](cfg.c.driver.md#cfg_show_causes)
    - [`cmd_free_argv`](cmd.c.driver.md#cmd_free_argv)


