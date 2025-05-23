# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file primarily defines and manages command structures and operations within tmux. It includes a comprehensive list of command entries, each representing a specific command that tmux can execute, such as attaching a session, binding keys, managing panes, and handling buffers. These command entries are stored in an array, `cmd_table`, which serves as a registry for all available commands in tmux.

The file also provides a variety of utility functions for handling command arguments and command lists. These functions include operations for logging, appending, and packing argument vectors, as well as parsing and copying commands. Additionally, the file defines structures and functions for managing command lists, which are collections of commands that can be executed in sequence. The code also includes functionality for handling mouse events within tmux, such as determining the current mouse position relative to a pane or window. Overall, this file is crucial for the command processing and execution logic in tmux, enabling users to interact with and control their terminal sessions effectively.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/time.h`
- `fnmatch.h`
- `pwd.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### cmd_attach_session_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_attach_session_entry` is a global constant variable that represents a command entry structure for attaching to a session in the tmux terminal multiplexer. It is part of a larger command table that defines various commands available in the application, allowing users to manage sessions effectively.
- **Use**: This variable is used to reference the command entry associated with the 'attach session' command in the command processing logic.


---
### cmd_bind_key_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_bind_key_entry` is a global constant variable that represents a command entry structure used in the command processing system of the application. It is part of a larger set of command entries that define various commands available to the user, specifically for binding keys to actions.
- **Use**: This variable is used to reference the command entry associated with binding keys in the command table.


---
### cmd_break_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_break_pane_entry` is a global constant variable that represents a command entry structure for breaking a pane in a terminal multiplexer. It is part of a larger command table that defines various commands available in the application, allowing for modular command handling.
- **Use**: This variable is used to reference the command entry associated with breaking a pane, enabling its functionality within the command processing system.


---
### cmd_capture_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_capture_pane_entry` is a global constant variable that represents a command entry structure for capturing the current pane in a terminal multiplexer application. It is part of a larger command table that defines various commands available in the application, each associated with specific functionality.
- **Use**: This variable is used to reference the command entry for capturing a pane, allowing it to be invoked as part of the command processing system.


---
### cmd_choose_buffer_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_choose_buffer_entry` is a global constant variable that represents a command entry structure for choosing a buffer in the command interface. It is part of a larger set of command entries that define various commands available in the application, likely related to a terminal multiplexer or similar tool.
- **Use**: This variable is used to reference the specific command entry for choosing a buffer, allowing it to be included in command tables or invoked as part of command processing.


---
### cmd_choose_client_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_choose_client_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically related to choosing a client. This structure likely contains function pointers and metadata necessary for executing the command associated with client selection.
- **Use**: This variable is used to reference the command entry for choosing a client within the command processing framework.


---
### cmd_choose_tree_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_choose_tree_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a collection of command entries that define various commands available in the application, specifically related to choosing a tree structure.
- **Use**: This variable is used to reference the command entry for the 'choose tree' command within the command processing framework.


---
### cmd_clear_history_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_clear_history_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a collection of command entries that define various commands available in the application, specifically for clearing the command history.
- **Use**: This variable is used to reference the command entry for clearing history in the command table.


---
### cmd_clear_prompt_history_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_clear_prompt_history_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically for clearing the prompt history.
- **Use**: This variable is used to reference the command entry associated with clearing the prompt history in the command table.


---
### cmd_clock_mode_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_clock_mode_entry` is a global constant variable that represents a command entry structure specifically for the clock mode functionality within the application. This structure likely contains information about the command's name, its associated function, and any flags or parameters relevant to its execution.
- **Use**: This variable is used to define and register the clock mode command in the command table for the application.


---
### cmd_command_prompt_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_command_prompt_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically for handling command prompts.
- **Use**: This variable is used to reference the command entry associated with the command prompt functionality within the application.


---
### cmd_confirm_before_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_confirm_before_entry` is a global variable that represents a command entry structure, which is used to define a specific command within the command framework of the application. This structure likely contains information such as the command's name, its associated function, and any flags or parameters relevant to its execution.
- **Use**: This variable is used to reference the command entry for confirming actions before they are executed.


---
### cmd_copy_mode_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_copy_mode_entry` is a global constant variable that represents a command entry structure for the 'copy mode' functionality in the application. It is part of a larger set of command entries that define various commands available in the system, each encapsulated in a `cmd_entry` structure.
- **Use**: This variable is used to reference the specific command entry associated with the copy mode functionality within the command processing system.


---
### cmd_customize_mode_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_customize_mode_entry` is a global constant variable that represents a command entry structure used in the command processing system of the application. It is part of a larger set of command entries that define various commands available to the user, specifically related to customizing modes within the application.
- **Use**: This variable is used to reference the command entry for customizing modes in the command table.


---
### cmd_delete_buffer_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_delete_buffer_entry` is a global constant variable that represents a command entry structure for the 'delete buffer' command in the application. It is part of a larger command table that defines various commands available in the system, allowing for structured command handling.
- **Use**: This variable is used to reference the specific command entry associated with deleting a buffer, facilitating command execution and management.


---
### cmd_detach_client_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_detach_client_entry` is a global constant variable that represents a command entry structure for detaching a client in the application. It is part of a larger command table that defines various commands available in the system, each represented by a `cmd_entry` structure.
- **Use**: This variable is used to reference the specific command entry for detaching a client, allowing it to be included in command processing and execution.


---
### cmd_display_menu_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_display_menu_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a collection of command entries that define various commands available in the application, specifically related to displaying a menu.
- **Use**: This variable is used to reference the command entry for displaying a menu in the command processing logic.


---
### cmd_display_message_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_display_message_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a collection of command entries that define various commands available in the application, specifically for displaying messages to the user.
- **Use**: This variable is used to reference the command entry associated with displaying messages in the command table.


---
### cmd_display_popup_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_display_popup_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command for displaying a popup in the application. This structure likely contains information such as the command name, its associated function, and any flags or parameters relevant to the command's execution.
- **Use**: This variable is used to register and reference the command associated with displaying a popup in the command handling system.


---
### cmd_display_panes_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_display_panes_entry` is a global constant variable that represents a command entry structure for displaying panes in a terminal multiplexer application. It is part of a larger command table that defines various commands available to the user, encapsulating the command's name, arguments, and associated functionality.
- **Use**: This variable is used to reference the command entry for displaying panes, allowing it to be accessed and executed within the command processing logic of the application.


---
### cmd_find_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_find_window_entry` is a global constant variable that represents a command entry structure for the 'find window' command in the application. It is part of a larger command table that defines various commands available in the system, allowing for structured command handling.
- **Use**: This variable is used to reference the specific command entry associated with the 'find window' command within the command processing framework.


---
### cmd_has_session_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_has_session_entry` is a global constant variable that represents a command entry structure for handling session-related commands in the application. It is part of a larger command table that defines various commands available in the system, allowing for structured command processing.
- **Use**: This variable is used to reference a specific command entry related to session management within the command processing framework.


---
### cmd_if_shell_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_if_shell_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a collection of command entries that define various commands available in the application, specifically related to shell command execution.
- **Use**: This variable is used to reference the command entry for the 'if-shell' command within the command processing framework.


---
### cmd_join_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_join_pane_entry` is a global constant variable that represents a command entry structure for joining panes in a terminal multiplexer application. It is part of a larger command table that defines various commands available to the user, encapsulating the command's properties and behavior.
- **Use**: This variable is used to reference the command entry for the 'join pane' operation within the command handling system.


---
### cmd_kill_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_kill_pane_entry` is a global constant variable that represents a command entry structure specifically for the 'kill pane' command in the application. This structure likely contains information such as the command's name, its associated function, and any flags or usage details relevant to executing the command.
- **Use**: This variable is used to define the behavior and properties of the 'kill pane' command within the command handling system.


---
### cmd_kill_server_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_kill_server_entry` is a global constant variable that represents a command entry structure specifically designed for the 'kill server' command in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments that pertain to its execution.
- **Use**: This variable is used to define the behavior and properties of the 'kill server' command within the command handling system.


---
### cmd_kill_session_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_kill_session_entry` is a global constant variable that represents a command entry structure specifically for the 'kill session' command in the application. This structure likely contains details such as the command's name, its associated arguments, and any flags that dictate its behavior within the command processing system.
- **Use**: This variable is used to define the properties and behavior of the 'kill session' command in the command table.


---
### cmd_kill_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_kill_window_entry` is a global constant variable that represents a command entry structure specifically for the 'kill window' command in the application. This structure likely contains function pointers and metadata related to the command's execution, such as its name, usage, and associated flags.
- **Use**: This variable is used to define the behavior and properties of the 'kill window' command within the command processing system.


---
### cmd_last_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_last_pane_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to the last pane in a terminal multiplexer context. This structure likely contains information such as the command's name, its associated function, and any flags or options relevant to its execution.
- **Use**: This variable is used to reference the command entry for the last pane, allowing the command system to access its properties and behavior.


---
### cmd_last_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_last_window_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to the last window in a terminal multiplexer context. This structure likely contains information such as the command's name, its associated function, and any flags or arguments relevant to its execution.
- **Use**: This variable is used to access the command entry for the last window, allowing the command to be executed or referenced within the command processing system.


---
### cmd_link_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_link_window_entry` is a global constant variable that represents a command entry structure for linking a window in the command interface. It is part of a larger command table that defines various commands available in the application, specifically related to window management.
- **Use**: `cmd_link_window_entry` is used to reference the command entry associated with linking a window, allowing the command parser to identify and execute this specific command when invoked.


---
### cmd_list_buffers_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_list_buffers_entry` is a global constant variable that represents a command entry structure specifically for listing buffers in the application. It is part of a larger command table that defines various commands available in the system, allowing for organized command handling and execution.
- **Use**: `cmd_list_buffers_entry` is used as a reference in the command table to facilitate the execution of the command that lists all buffers.


---
### cmd_list_clients_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_list_clients_entry` is a global constant variable that represents a command entry structure specifically for listing clients in the application. It is part of a larger set of command entries that define various commands available in the system, allowing for structured command handling.
- **Use**: This variable is used to reference the command entry associated with listing clients, facilitating command execution and management within the application.


---
### cmd_list_commands_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_list_commands_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command in the command list for a command-line interface. This structure likely contains details such as the command name, its associated function, and any flags or options related to the command.
- **Use**: This variable is used to provide access to the command entry for listing commands in the command-line interface.


---
### cmd_list_keys_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_list_keys_entry` is a global constant variable that represents a command entry structure for listing key bindings in the application. It is part of a larger set of command entries that define various commands available in the system, allowing for structured command handling and execution.
- **Use**: This variable is used to reference the command entry associated with listing key bindings, facilitating command processing in the application.


---
### cmd_list_panes_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_list_panes_entry` is a global constant variable that represents a command entry structure specifically for listing panes in a terminal multiplexer context. It is part of a larger command table that defines various commands available in the application, allowing for structured command handling and execution.
- **Use**: This variable is used to reference the command entry associated with listing panes, facilitating command parsing and execution within the application.


---
### cmd_list_sessions_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_list_sessions_entry` is a global constant variable that represents a command entry structure specifically for listing sessions in a command-line interface. It is part of a larger command table that defines various commands available in the application, allowing for structured command handling.
- **Use**: This variable is used to reference the command entry associated with listing sessions, facilitating command execution and management within the application.


---
### cmd_list_windows_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_list_windows_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to listing windows in the application. This structure likely contains information such as the command name, its associated function, and any flags or parameters relevant to its execution.
- **Use**: This variable is used to provide a command entry for listing windows in the command table of the application.


---
### cmd_load_buffer_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_load_buffer_entry` is a global variable that represents a command entry structure used in a command table. It is defined as a constant, indicating that its value should not change after initialization, and it is likely used to facilitate the loading of buffers in the context of a command-line interface or terminal multiplexer.
- **Use**: This variable is used to reference a specific command entry related to loading buffers within the command processing system.


---
### cmd_lock_client_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_lock_client_entry` is a global constant variable that represents a command entry structure specifically for locking a client in the context of a command processing system. It is part of a larger set of command entries that define various commands available in the application, allowing for structured command handling.
- **Use**: This variable is used to reference the command entry associated with locking a client, facilitating command execution and management within the application.


---
### cmd_lock_server_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_lock_server_entry` is a global constant variable that represents a command entry structure for locking the server in the context of a command processing system. It is part of a larger command table that defines various commands available in the application.
- **Use**: `cmd_lock_server_entry` is used to reference the specific command entry associated with the lock server command, allowing the command processing system to execute the appropriate functionality when this command is invoked.


---
### cmd_lock_session_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_lock_session_entry` is a global constant variable that represents a command entry structure for locking a session in the application. It is part of a larger command table that defines various commands available in the system, allowing for structured command handling.
- **Use**: This variable is used to reference the command entry associated with locking a session, facilitating command execution and management within the application.


---
### cmd_move_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_move_pane_entry` is a global constant variable that represents a command entry structure specifically for the 'move pane' command in the application. This structure likely contains details such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to define the behavior and properties of the 'move pane' command within the command handling system.


---
### cmd_move_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_move_window_entry` is a global constant variable that represents a command entry structure specifically for the 'move window' command in the application. This structure likely contains function pointers and metadata related to the command's execution, usage, and options.
- **Use**: This variable is used to define the behavior and properties of the 'move window' command within the command processing system.


---
### cmd_new_session_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_new_session_entry` is a global constant variable that represents a command entry structure for creating a new session in the application. It is part of a command table that defines various commands available in the system, allowing for structured command handling.
- **Use**: This variable is used to reference the command entry associated with the creation of a new session, facilitating command execution and management.


---
### cmd_new_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_new_window_entry` is a global constant variable that represents a command entry structure specifically for creating a new window in the application. It is part of a larger command table that defines various commands available in the system, allowing for modular command handling.
- **Use**: This variable is used to reference the command entry for creating a new window, facilitating command execution within the application.


---
### cmd_next_layout_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_next_layout_entry` is a global constant variable that represents a specific command entry in a command table for a terminal multiplexer application. It is defined as a pointer to a `struct cmd_entry`, which likely contains information about the command's name, function, and associated properties.
- **Use**: This variable is used to reference the command associated with moving to the next layout in the command processing system.


---
### cmd_next_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_next_window_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command for navigating to the next window in a terminal multiplexer context. This structure likely contains details such as the command name, its associated function, and any flags or arguments required for execution.
- **Use**: This variable is used to facilitate the execution of the command that allows users to switch to the next window within the application.


---
### cmd_paste_buffer_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_paste_buffer_entry` is a global constant variable that represents a command entry structure specifically for the paste buffer command in the application. This structure likely contains information about the command's name, its associated function, and any flags or arguments it may require.
- **Use**: `cmd_paste_buffer_entry` is used to define the behavior and properties of the paste buffer command within the command handling system of the application.


---
### cmd_pipe_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_pipe_pane_entry` is a global constant variable that represents a command entry structure for the 'pipe pane' command in the application. This structure likely contains function pointers and metadata related to the command's execution, usage, and options.
- **Use**: It is used as part of a command table to facilitate command parsing and execution within the application.


---
### cmd_previous_layout_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_previous_layout_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to navigating to the previous layout in a command-line interface. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to provide access to the command entry for the 'previous layout' command within the command processing system.


---
### cmd_previous_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_previous_window_entry` is a global constant variable that represents a command entry structure for navigating to the previous window in a command-line interface. It is part of a larger command table that defines various commands available in the application.
- **Use**: `cmd_previous_window_entry` is used to facilitate the execution of the command that allows users to switch to the previous window within the application.


---
### cmd_refresh_client_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_refresh_client_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically related to refreshing client entries.
- **Use**: This variable is used to reference the command entry for refreshing client information within the command processing framework.


---
### cmd_rename_session_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_rename_session_entry` is a global constant variable that represents a command entry structure specifically for renaming a session in the application. It is part of a larger command table that defines various commands available in the system, allowing for structured command handling.
- **Use**: This variable is used to reference the command entry associated with the rename session functionality within the command processing system.


---
### cmd_rename_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_rename_window_entry` is a global constant variable that represents a command entry structure specifically for the 'rename window' command in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments that pertain to the command's execution.
- **Use**: This variable is used to define the behavior and properties of the 'rename window' command within the command processing system.


---
### cmd_resize_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_resize_pane_entry` is a global constant variable that represents a command entry structure specifically for resizing a pane in a terminal multiplexer application. It is part of a larger set of command entries that define various operations that can be performed within the application.
- **Use**: This variable is used to provide the command entry details for the resize pane operation, allowing it to be referenced in command parsing and execution.


---
### cmd_resize_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_resize_window_entry` is a global constant variable that represents a command entry structure specifically for resizing a window in the application. It is part of a larger command table that defines various commands available in the system, allowing for modular command handling.
- **Use**: This variable is used to reference the command entry associated with the window resizing functionality within the command processing system.


---
### cmd_respawn_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_respawn_pane_entry` is a global constant variable that represents a command entry structure for respawning a pane in the terminal multiplexer. It is part of a larger command table that defines various commands available in the application, allowing for structured command handling.
- **Use**: This variable is used to reference the command entry associated with the respawn pane command in the command processing logic.


---
### cmd_respawn_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_respawn_window_entry` is a global constant variable that represents a command entry structure for the 'respawn window' command in the application. This structure likely contains function pointers and metadata related to the command's execution, such as its name, usage, and associated flags.
- **Use**: This variable is used to define the behavior and properties of the 'respawn window' command within the command handling system.


---
### cmd_rotate_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_rotate_window_entry` is a global constant variable that represents a command entry structure for the 'rotate window' command in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to define the behavior and properties of the 'rotate window' command within the command processing system.


---
### cmd_run_shell_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_run_shell_entry` is a global constant variable that represents a command entry structure for executing shell commands within the application. It is part of a larger command table that defines various command entries, each associated with specific functionalities in the application.
- **Use**: This variable is used to reference the command entry for running shell commands, allowing the application to execute shell commands as part of its command processing.


---
### cmd_save_buffer_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_save_buffer_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command for saving a buffer in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments related to the command.
- **Use**: This variable is used to register the save buffer command in the command table, allowing it to be invoked by the user.


---
### cmd_select_layout_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_select_layout_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to selecting a layout in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to provide a command entry for selecting a layout in the command processing system.


---
### cmd_select_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_select_pane_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to selecting a pane in a terminal multiplexer context. This structure likely contains information such as the command's name, its associated arguments, and any flags that dictate its behavior.
- **Use**: This variable is used to provide a specific command entry for selecting a pane, which can be referenced in command parsing and execution within the application.


---
### cmd_select_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_select_window_entry` is a global constant variable that represents a command entry structure specifically for selecting a window in the application. It is part of a larger command table that defines various commands available in the system, allowing for structured command handling.
- **Use**: This variable is used to reference the command entry associated with the action of selecting a window within the command processing framework.


---
### cmd_send_keys_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_send_keys_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to sending keys in the application. This structure likely contains information such as the command name, its associated function, and any flags or arguments that pertain to the command's execution.
- **Use**: This variable is used to register and access the command for sending keys within the command processing system.


---
### cmd_send_prefix_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_send_prefix_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically related to sending prefix commands.
- **Use**: This variable is used to reference the command entry for sending prefix commands within the command processing logic.


---
### cmd_server_access_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_server_access_entry` variable is a global constant that represents a command entry structure used within the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically related to server access functionalities.
- **Use**: This variable is used to reference the command entry for server access in the command table, allowing the application to handle server access commands appropriately.


---
### cmd_set_buffer_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_set_buffer_entry` is a global constant variable that represents a command entry structure for setting a buffer in the command processing system. It is part of a larger set of command entries that define various commands available in the application, allowing for structured command handling.
- **Use**: This variable is used to reference the command entry associated with the 'set buffer' command in the command table.


---
### cmd_set_environment_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_set_environment_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to setting environment variables in the application. This structure likely contains information such as the command name, its associated function, and any flags or arguments that pertain to the command's behavior.
- **Use**: This variable is used to register and provide access to the command for setting environment variables within the command processing system.


---
### cmd_set_hook_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_set_hook_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically related to setting hooks.
- **Use**: This variable is used to reference the command entry for setting hooks within the command processing framework.


---
### cmd_set_option_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_set_option_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically for setting options.
- **Use**: This variable is used to reference the command entry for setting options within the command processing framework.


---
### cmd_set_window_option_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_set_window_option_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to setting window options in the application. This structure likely contains information such as the command name, its arguments, and associated flags that dictate its behavior within the command processing system.
- **Use**: This variable is used to register and identify the command for setting window options in the command table.


---
### cmd_show_buffer_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_show_buffer_entry` is a global constant variable that represents a command entry structure for displaying buffer information in the application. It is part of a larger command table that defines various commands available in the system, allowing for structured command handling and execution.
- **Use**: This variable is used to reference the command entry associated with showing buffer details, facilitating command parsing and execution.


---
### cmd_show_environment_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_show_environment_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to displaying environment variables in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to register and access the command for showing environment entries within the command processing system.


---
### cmd_show_hooks_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_show_hooks_entry` is a global constant variable that represents a command entry structure, specifically designed to handle the display of hooks in the command interface. It is part of a larger command table that defines various command entries for a command-line interface, allowing for modular command handling.
- **Use**: This variable is used to reference the command entry associated with showing hooks in the command processing system.


---
### cmd_show_messages_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_show_messages_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to displaying messages in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to register and access the command for showing messages within the command processing system.


---
### cmd_show_options_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_show_options_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically for showing options.
- **Use**: This variable is used to reference the command entry for displaying options in the command table.


---
### cmd_show_prompt_history_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_show_prompt_history_entry` is a global constant variable that represents a command entry structure specifically designed to handle the display of prompt history in a command-line interface. This structure likely contains function pointers and metadata related to the command's behavior and usage.
- **Use**: This variable is used to reference the command entry for showing prompt history within the command processing system.


---
### cmd_show_window_options_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_show_window_options_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to displaying window options in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to register and provide access to the command for showing window options within the command handling system.


---
### cmd_source_file_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_source_file_entry` is a global constant variable that represents a command entry structure, which is likely used to define a specific command in the command processing system of the application. This structure is part of a larger command table that holds various command entries, each associated with specific functionalities within the application.
- **Use**: This variable is used to reference a command entry for the 'source file' command in the command processing system.


---
### cmd_split_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_split_window_entry` is a global constant variable that represents a command entry structure for splitting a window in the application. It is part of a larger command table that defines various commands available in the system, allowing for modular command handling.
- **Use**: This variable is used to reference the command entry associated with the split window functionality within the command processing system.


---
### cmd_start_server_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_start_server_entry` is a global constant variable that represents a command entry structure for starting a server in the application. It is part of a larger command table that defines various commands available in the system, each associated with specific functionalities.
- **Use**: This variable is used to reference the command entry for starting a server, allowing it to be included in command processing and execution logic.


---
### cmd_suspend_client_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_suspend_client_entry` is a global constant variable that represents a command entry structure for suspending a client in the application. It is part of a larger command table that defines various commands available in the system, each associated with specific functionalities.
- **Use**: This variable is used to reference the command entry for suspending a client, allowing the application to execute the corresponding command when invoked.


---
### cmd_swap_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_swap_pane_entry` is a global constant variable that represents a command entry structure for swapping panes in a terminal multiplexer application. It is part of a larger command table that defines various commands available to the user, encapsulating the command's name, arguments, and associated functionality.
- **Use**: This variable is used to provide the command entry details for the 'swap pane' command, allowing it to be referenced and executed within the command processing logic.


---
### cmd_swap_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_swap_window_entry` is a global constant variable that represents a command entry structure for the 'swap window' command in the application. This structure likely contains information about the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to define the behavior and properties of the 'swap window' command within the command processing system.


---
### cmd_switch_client_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_switch_client_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically for switching between clients.
- **Use**: `cmd_switch_client_entry` is utilized in the command table to facilitate the execution of the switch client command within the application.


---
### cmd_unbind_key_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_unbind_key_entry` is a global constant variable that represents a command entry structure for unbinding a key in the command interface. It is part of a larger set of command entries that define various commands available in the application, likely related to a terminal multiplexer or similar software. This structure is used to encapsulate the details of the command, such as its name, arguments, and associated functionality.
- **Use**: This variable is used to reference the command entry for unbinding keys within the command processing system.


---
### cmd_unlink_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_unlink_window_entry` is a global constant variable that represents a command entry structure for unlinking a window in the application. It is part of a command table that defines various commands available in the system, allowing for structured command handling.
- **Use**: This variable is used to reference the command entry associated with the unlink window operation in the command processing logic.


---
### cmd_wait_for_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_wait_for_entry` is a global variable that holds a reference to a `cmd_entry` structure, which defines a command that the application can execute. This structure likely contains information such as the command's name, its associated function, and any flags or parameters relevant to its execution.
- **Use**: This variable is used to register and identify a specific command within the command processing system of the application.


---
### cmd_table
- **Type**: `array of pointers to `struct cmd_entry``
- **Description**: The `cmd_table` variable is a global array that holds pointers to various command entries, each represented by a `struct cmd_entry`. This array serves as a centralized registry of commands that can be executed within the application, allowing for easy access and management of command functionalities.
- **Use**: It is used to look up command entries by name during command parsing and execution.


---
### cmd_list_next_group
- **Type**: `u_int`
- **Description**: `cmd_list_next_group` is a static global variable that holds the next group number for new command lists. It is initialized to 1 and is used to uniquely identify groups of commands within the command list structure.
- **Use**: This variable is incremented each time a new command list is created, ensuring that each list has a unique group identifier.


---
### i
- **Type**: `int`
- **Description**: The variable `i` is an integer used as an index to iterate over the elements of an argument vector (`argv`). It is typically employed in loops to access each argument passed to a command-line program.
- **Use**: The variable `i` is used in the `cmd_log_argv` function to log each argument in the `argv` array.


---
### argv
- **Type**: `char **`
- **Description**: `argv` is a pointer to an array of character pointers (strings) that represent the command-line arguments passed to the program. It is typically used in the context of the `main` function to access the arguments provided by the user when executing the program.
- **Use**: `argv` is used to log the command-line arguments in the `cmd_log_argv` function.


# Data Structures

---
### cmd
- **Type**: `struct`
- **Members**:
    - `entry`: A pointer to a constant `cmd_entry` structure, representing the command's entry point.
    - `args`: A pointer to an `args` structure, holding the arguments for the command.
    - `group`: An unsigned integer representing the group number of the command.
    - `file`: A pointer to a character string indicating the source file where the command is defined.
    - `line`: An unsigned integer indicating the line number in the source file where the command is defined.
    - `qentry`: A TAILQ_ENTRY structure for linking commands in a queue.
- **Description**: The `cmd` structure represents an instance of a command in the system, encapsulating the command's entry point, arguments, and metadata such as the source file and line number. It also includes a group identifier for organizing commands and a queue entry for managing command lists. This structure is part of a larger command management system, likely used in a terminal multiplexer or similar application, where commands are parsed, executed, and managed in a structured manner.


# Functions

---
### cmd_prepend_argv<!-- {{#callable:cmd_prepend_argv}} -->
The `cmd_prepend_argv` function adds a new argument to the beginning of an existing argument vector, updating the argument count accordingly.
- **Inputs**:
    - `argc`: A pointer to an integer representing the number of arguments in the argument vector.
    - `argv`: A pointer to a pointer to a character array, representing the argument vector.
    - `arg`: A constant character pointer representing the new argument to be prepended to the argument vector.
- **Control Flow**:
    - Allocate memory for a new argument vector that is one element larger than the current vector using `xreallocarray`.
    - Copy the new argument `arg` into the first position of the new argument vector using `xstrdup`.
    - Iterate over the existing argument vector and copy each element to the new argument vector, starting from the second position.
    - Free the memory allocated for the old argument vector using `free`.
    - Update the original argument vector pointer to point to the new argument vector.
    - Increment the argument count by one.
- **Output**:
    - The function does not return a value, but it modifies the input argument vector to include the new argument at the beginning and updates the argument count.


---
### cmd_append_argv<!-- {{#callable:cmd_append_argv}} -->
The `cmd_append_argv` function appends a new argument to an existing argument vector, reallocating memory as necessary.
- **Inputs**:
    - `argc`: A pointer to an integer representing the current number of arguments in the argument vector.
    - `argv`: A pointer to a pointer to a character array, representing the argument vector.
    - `arg`: A constant character pointer representing the new argument to be appended to the argument vector.
- **Control Flow**:
    - The function reallocates the memory for the argument vector to accommodate one more argument using `xreallocarray`.
    - The new argument is duplicated using `xstrdup` and added to the end of the argument vector.
    - The argument count (`argc`) is incremented to reflect the addition of the new argument.
- **Output**:
    - The function does not return a value; it modifies the argument vector and count in place.


---
### cmd_pack_argv<!-- {{#callable:cmd_pack_argv}} -->
The `cmd_pack_argv` function concatenates an array of argument strings into a single buffer, ensuring the buffer size is not exceeded.
- **Inputs**:
    - `argc`: The number of arguments in the `argv` array.
    - `argv`: An array of strings representing the arguments to be packed into the buffer.
    - `buf`: A character buffer where the concatenated arguments will be stored.
    - `len`: The size of the buffer `buf`.
- **Control Flow**:
    - If `argc` is zero, the function returns 0 immediately, indicating no arguments to pack.
    - The function logs the arguments using `cmd_log_argv`.
    - The buffer `buf` is initialized to an empty string.
    - A loop iterates over each argument in `argv`.
    - For each argument, `strlcpy` is used to copy the argument into `buf`, checking if the buffer size is exceeded.
    - If the buffer size is exceeded during any copy, the function returns -1, indicating an error.
    - The length of the current argument plus one (for the null terminator) is calculated and subtracted from `len`, and `buf` is incremented by this length to point to the next position in the buffer.
    - If all arguments are successfully packed, the function returns 0.
- **Output**:
    - Returns 0 on success, or -1 if the buffer size is exceeded during packing.


---
### cmd_unpack_argv<!-- {{#callable:cmd_unpack_argv}} -->
The `cmd_unpack_argv` function unpacks a buffer containing packed arguments into an argument vector.
- **Inputs**:
    - `buf`: A character buffer containing packed arguments, with each argument separated by a null character.
    - `len`: The length of the buffer, indicating the total size of the packed data.
    - `argc`: The number of arguments expected to be unpacked from the buffer.
    - `argv`: A pointer to a pointer to a character array, where the unpacked arguments will be stored.
- **Control Flow**:
    - Check if `argc` is zero; if so, return 0 immediately as there are no arguments to unpack.
    - Allocate memory for the argument vector `argv` using `xcalloc`, with space for `argc` pointers.
    - Ensure the buffer is null-terminated by setting the last character to '\0'.
    - Iterate over the number of arguments (`argc`), unpacking each argument from the buffer.
    - For each argument, check if the remaining buffer length is zero; if so, free the allocated memory and return -1 indicating an error.
    - Calculate the length of the current argument using `strlen` and copy it to the argument vector using `xstrdup`.
    - Advance the buffer pointer and decrease the remaining length by the length of the current argument plus one (for the null terminator).
    - Log the unpacked arguments using `cmd_log_argv`.
    - Return 0 to indicate successful unpacking.
- **Output**:
    - Returns 0 on success, or -1 if there is an error due to insufficient buffer length.
- **Functions called**:
    - [`cmd_free_argv`](#cmd_free_argv)


---
### cmd_copy_argv<!-- {{#callable:cmd_copy_argv}} -->
The `cmd_copy_argv` function duplicates an argument vector, ensuring it is terminated by a NULL pointer.
- **Inputs**:
    - `argc`: The number of arguments in the argument vector.
    - `argv`: The argument vector, which is an array of strings.
- **Control Flow**:
    - Check if `argc` is zero; if so, return NULL immediately as there are no arguments to copy.
    - Allocate memory for a new argument vector `new_argv` with `argc + 1` elements to include a NULL terminator.
    - Iterate over each element in the `argv` array up to `argc`.
    - For each non-NULL element in `argv`, duplicate the string using `xstrdup` and assign it to the corresponding position in `new_argv`.
    - Return the newly created `new_argv` array.
- **Output**:
    - A newly allocated argument vector that is a copy of the input `argv`, terminated by a NULL pointer.


---
### cmd_free_argv<!-- {{#callable:cmd_free_argv}} -->
The `cmd_free_argv` function deallocates memory for an argument vector and its elements.
- **Inputs**:
    - `argc`: The number of elements in the argument vector.
    - `argv`: A pointer to an array of strings (argument vector) to be freed.
- **Control Flow**:
    - Check if `argc` is zero; if so, return immediately as there is nothing to free.
    - Iterate over each element in the `argv` array, freeing each string individually using `free()`.
    - After all elements are freed, free the `argv` array itself.
- **Output**:
    - This function does not return any value; it performs memory deallocation for the provided argument vector.


---
### cmd_stringify_argv<!-- {{#callable:cmd_stringify_argv}} -->
The `cmd_stringify_argv` function converts an argument vector into a single space-separated string, escaping each argument as necessary.
- **Inputs**:
    - `argc`: The number of arguments in the argument vector.
    - `argv`: An array of strings representing the argument vector to be converted into a single string.
- **Control Flow**:
    - If `argc` is zero, the function returns an empty string using `xstrdup`.
    - Iterates over each argument in `argv`, escaping it using `args_escape`.
    - Logs the escaped argument using `log_debug`.
    - Calculates the new length required for the buffer to accommodate the escaped argument and a space, then reallocates the buffer using `xrealloc`.
    - If it's the first argument, initializes the buffer with an empty string; otherwise, appends a space to the buffer.
    - Appends the escaped argument to the buffer using `strlcat`.
    - Frees the memory allocated for the escaped argument.
    - Returns the final concatenated string.
- **Output**:
    - Returns a newly allocated string containing the space-separated, escaped arguments from the input vector.


---
### cmd_get_entry<!-- {{#callable:cmd_get_entry}} -->
The `cmd_get_entry` function retrieves the command entry associated with a given command structure.
- **Inputs**:
    - `cmd`: A pointer to a `struct cmd` which contains information about a command, including its associated entry.
- **Control Flow**:
    - The function directly returns the `entry` member of the `cmd` structure passed to it.
- **Output**:
    - A pointer to a `struct cmd_entry`, which represents the command entry associated with the given command.


---
### cmd_get_args<!-- {{#callable:cmd_get_args}} -->
The `cmd_get_args` function retrieves the arguments associated with a given command structure.
- **Inputs**:
    - `cmd`: A pointer to a `struct cmd` which represents a command instance containing various attributes including its arguments.
- **Control Flow**:
    - The function directly accesses the `args` member of the `cmd` structure and returns it.
- **Output**:
    - A pointer to a `struct args`, which contains the arguments associated with the provided command.


---
### cmd_get_group<!-- {{#callable:cmd_get_group}} -->
The `cmd_get_group` function retrieves the group number associated with a given command structure.
- **Inputs**:
    - `cmd`: A pointer to a `struct cmd` which represents a command instance containing various attributes including the group number.
- **Control Flow**:
    - The function takes a single argument, a pointer to a `struct cmd`.
    - It accesses the `group` member of the `cmd` structure.
    - The function returns the value of the `group` member.
- **Output**:
    - The function returns an unsigned integer (`u_int`) representing the group number of the command.


---
### cmd_get_source<!-- {{#callable:cmd_get_source}} -->
The `cmd_get_source` function retrieves the source file name and line number associated with a given command structure.
- **Inputs**:
    - `cmd`: A pointer to a `struct cmd` which contains information about a command, including its source file and line number.
    - `file`: A pointer to a `const char*` where the function will store the file name associated with the command, if not NULL.
    - `line`: A pointer to a `u_int` where the function will store the line number associated with the command, if not NULL.
- **Control Flow**:
    - Check if the `file` pointer is not NULL; if true, assign the `cmd->file` value to `*file`.
    - Check if the `line` pointer is not NULL; if true, assign the `cmd->line` value to `*line`.
- **Output**:
    - The function does not return a value; it outputs the file name and line number through the provided pointers if they are not NULL.


---
### cmd_get_alias<!-- {{#callable:cmd_get_alias}} -->
The `cmd_get_alias` function retrieves the alias for a given command name from the global options if it exists.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the command for which the alias is being searched.
- **Control Flow**:
    - Retrieve the 'command-alias' options entry from global options.
    - If the options entry is not found, return NULL.
    - Calculate the length of the input command name.
    - Iterate over each item in the options array associated with the 'command-alias' entry.
    - For each item, retrieve its value and locate the '=' character in the string.
    - If '=' is found, calculate the length of the substring before '=' and compare it with the length of the input name.
    - If the lengths match and the substrings are equal, return a duplicate of the string following '=' as the alias.
    - If no matching alias is found after iterating through all items, return NULL.
- **Output**:
    - Returns a dynamically allocated string containing the alias for the command if found, or NULL if no alias is found.


---
### cmd_find<!-- {{#callable:cmd_find}} -->
The `cmd_find` function searches for a command entry by its name in a predefined command table and handles cases of ambiguous or unknown commands.
- **Inputs**:
    - `name`: A string representing the name of the command to search for.
    - `cause`: A pointer to a string where an error message will be stored if the command is ambiguous or unknown.
- **Control Flow**:
    - Initialize `ambiguous` to 0 and `found` to NULL.
    - Iterate over the `cmd_table` array of command entries.
    - For each entry, check if the `alias` matches the `name`; if so, set `found` to this entry and break the loop.
    - If the `alias` does not match, check if the `name` partially matches the entry's `name` using `strncmp`.
    - If a partial match is found and `found` is already set, mark the command as ambiguous.
    - If a full match is found using `strcmp`, break the loop.
    - If the command is ambiguous, construct a list of possible matches and set an error message in `cause`.
    - If no command is found, set an error message in `cause` indicating the command is unknown.
    - Return the found command entry or NULL if ambiguous or unknown.
- **Output**:
    - Returns a pointer to the `cmd_entry` structure if a unique match is found, or NULL if the command is ambiguous or unknown.


---
### cmd_parse<!-- {{#callable:cmd_parse}} -->
The `cmd_parse` function parses a command from an argument vector and returns a command structure if successful, or NULL with an error message if not.
- **Inputs**:
    - `values`: An array of `struct args_value` representing the command arguments.
    - `count`: An unsigned integer representing the number of elements in the `values` array.
    - `file`: A string representing the file name where the command is being parsed, or NULL if not applicable.
    - `line`: An unsigned integer representing the line number in the file where the command is being parsed.
    - `cause`: A pointer to a string where an error message will be stored if the function fails.
- **Control Flow**:
    - Check if `count` is zero or if the first element of `values` is not of type `ARGS_STRING`; if so, set `cause` to "no command" and return NULL.
    - Use [`cmd_find`](#cmd_find) to locate the command entry corresponding to the first string in `values`; if not found, return NULL.
    - Parse the arguments using [`args_parse`](tmux.h.driver.md#args_parse) with the command entry's argument specifications; if parsing fails and `error` is NULL, set `cause` to the command's usage and return NULL.
    - If parsing fails and `error` is not NULL, set `cause` to the error message, free the error string, and return NULL.
    - Allocate memory for a new `cmd` structure, set its `entry` and `args` fields, and copy `file` and `line` if applicable.
    - Return the newly created `cmd` structure.
- **Output**:
    - Returns a pointer to a `struct cmd` representing the parsed command, or NULL if parsing fails, with an error message stored in `cause`.
- **Functions called**:
    - [`cmd_find`](#cmd_find)
    - [`args_parse`](tmux.h.driver.md#args_parse)


---
### cmd_free<!-- {{#callable:cmd_free}} -->
The `cmd_free` function deallocates memory associated with a `cmd` structure, including its file and arguments.
- **Inputs**:
    - `cmd`: A pointer to a `struct cmd` that represents a command instance to be freed.
- **Control Flow**:
    - The function first frees the memory allocated for the `file` member of the `cmd` structure using `free`.
    - It then calls `args_free` to deallocate the memory associated with the `args` member of the `cmd` structure.
    - Finally, it frees the memory allocated for the `cmd` structure itself.
- **Output**:
    - The function does not return any value; it performs memory deallocation for the given `cmd` structure.


---
### cmd_copy<!-- {{#callable:cmd_copy}} -->
The `cmd_copy` function creates a duplicate of a given command structure, including its entry, arguments, file, and line information.
- **Inputs**:
    - `cmd`: A pointer to the `cmd` structure that needs to be copied.
    - `argc`: An integer representing the number of arguments in the `argv` array.
    - `argv`: An array of strings representing additional arguments to be copied into the new command's arguments.
- **Control Flow**:
    - Allocate memory for a new `cmd` structure using `xcalloc` and assign it to `new_cmd`.
    - Copy the `entry` from the original `cmd` to `new_cmd`.
    - Copy the arguments from the original `cmd` using `args_copy`, passing `argc` and `argv` to include additional arguments, and assign them to `new_cmd->args`.
    - If the original `cmd` has a non-null `file`, duplicate the string using `xstrdup` and assign it to `new_cmd->file`.
    - Copy the `line` number from the original `cmd` to `new_cmd->line`.
    - Return the newly created `new_cmd` structure.
- **Output**:
    - Returns a pointer to the newly created `cmd` structure that is a copy of the input `cmd` with additional arguments.


---
### cmd_print<!-- {{#callable:cmd_print}} -->
The `cmd_print` function generates a string representation of a command by combining the command's name with its arguments.
- **Inputs**:
    - `cmd`: A pointer to a `struct cmd` which contains the command entry and its arguments.
- **Control Flow**:
    - Call `args_print` to get a string representation of the command's arguments.
    - Check if the returned string from `args_print` is not empty.
    - If not empty, use `xasprintf` to format the command name and arguments into a single string and assign it to `out`.
    - If empty, duplicate the command name using `xstrdup` and assign it to `out`.
    - Free the memory allocated for the arguments string `s`.
- **Output**:
    - Returns a dynamically allocated string that represents the command, which includes the command name and its arguments if any.


---
### cmd_list_new<!-- {{#callable:cmd_list_new}} -->
The `cmd_list_new` function creates and initializes a new command list structure.
- **Inputs**:
    - None
- **Control Flow**:
    - Allocate memory for a new `cmd_list` structure using `xcalloc` and assign it to `cmdlist`.
    - Initialize the `references` field of `cmdlist` to 1.
    - Assign a unique group number to `cmdlist->group` by incrementing `cmd_list_next_group`.
    - Allocate memory for the `list` field of `cmdlist` using `xcalloc`.
    - Initialize the `list` field as an empty tail queue using `TAILQ_INIT`.
    - Return the initialized `cmdlist` structure.
- **Output**:
    - A pointer to a newly allocated and initialized `cmd_list` structure.


---
### cmd_list_append<!-- {{#callable:cmd_list_append}} -->
The `cmd_list_append` function appends a command to the end of a command list, ensuring the command's group is set to the list's group.
- **Inputs**:
    - `cmdlist`: A pointer to a `struct cmd_list`, representing the list to which the command will be appended.
    - `cmd`: A pointer to a `struct cmd`, representing the command to be appended to the list.
- **Control Flow**:
    - Set the `group` attribute of the `cmd` to the `group` of the `cmdlist`.
    - Use the `TAILQ_INSERT_TAIL` macro to insert the `cmd` at the end of the `cmdlist->list`.
- **Output**:
    - The function does not return a value; it modifies the `cmdlist` by appending the `cmd` to it.


---
### cmd_list_append_all<!-- {{#callable:cmd_list_append_all}} -->
The `cmd_list_append_all` function appends all commands from one command list to another, updating the group of each command to match the destination list.
- **Inputs**:
    - `cmdlist`: A pointer to the destination `cmd_list` structure where commands will be appended.
    - `from`: A pointer to the source `cmd_list` structure from which commands will be taken.
- **Control Flow**:
    - Iterate over each command in the source list (`from`).
    - For each command, set its group to the group of the destination list (`cmdlist`).
    - Concatenate the source list (`from->list`) to the destination list (`cmdlist->list`) using `TAILQ_CONCAT`.
- **Output**:
    - The function does not return a value; it modifies the destination `cmd_list` in place by appending commands from the source list.


---
### cmd_list_move<!-- {{#callable:cmd_list_move}} -->
The `cmd_list_move` function transfers all commands from one command list to another and assigns a new group number to the destination list.
- **Inputs**:
    - `cmdlist`: A pointer to the destination `cmd_list` structure where commands will be moved.
    - `from`: A pointer to the source `cmd_list` structure from which commands will be moved.
- **Control Flow**:
    - The function uses `TAILQ_CONCAT` to concatenate the list of commands from the `from` list to the `cmdlist` list.
    - The `group` field of the `cmdlist` is updated to a new group number, which is incremented from `cmd_list_next_group`.
- **Output**:
    - The function does not return a value; it modifies the `cmdlist` and `from` structures in place.


---
### cmd_list_free<!-- {{#callable:cmd_list_free}} -->
The `cmd_list_free` function deallocates a command list and its associated resources if it is no longer referenced.
- **Inputs**:
    - `cmdlist`: A pointer to a `struct cmd_list` which represents the command list to be freed.
- **Control Flow**:
    - Decrement the reference count of the command list.
    - If the reference count is not zero, exit the function without freeing resources.
    - Iterate over each command in the command list using `TAILQ_FOREACH_SAFE`.
    - For each command, remove it from the list and free its resources using [`cmd_free`](#cmd_free).
    - Free the memory allocated for the command list's internal list structure.
    - Free the memory allocated for the command list itself.
- **Output**:
    - The function does not return a value; it performs cleanup operations on the command list.
- **Functions called**:
    - [`cmd_free`](#cmd_free)


---
### cmd_list_copy<!-- {{#callable:cmd_list_copy}} -->
The `cmd_list_copy` function creates a duplicate of a given command list, expanding any placeholders in the command arguments with provided values.
- **Inputs**:
    - `cmdlist`: A pointer to the original command list (`struct cmd_list`) that needs to be copied.
    - `argc`: An integer representing the number of arguments in the `argv` array.
    - `argv`: An array of strings (`char **`) containing arguments to replace placeholders in the command list.
- **Control Flow**:
    - The function begins by printing the original command list for debugging purposes.
    - A new command list is created using [`cmd_list_new`](#cmd_list_new).
    - The function iterates over each command in the original command list using `TAILQ_FOREACH`.
    - For each command, it checks if the command's group is different from the current group; if so, it updates the group in the new command list and increments the global group counter.
    - Each command is copied using [`cmd_copy`](#cmd_copy), with placeholders in arguments replaced by values from `argv`.
    - The copied command is appended to the new command list using [`cmd_list_append`](#cmd_list_append).
    - After processing all commands, the new command list is printed for debugging purposes.
    - Finally, the new command list is returned.
- **Output**:
    - Returns a pointer to the newly created command list (`struct cmd_list *`) that is a copy of the original, with arguments expanded.
- **Functions called**:
    - [`cmd_list_print`](#cmd_list_print)
    - [`cmd_list_new`](#cmd_list_new)
    - [`cmd_copy`](#cmd_copy)
    - [`cmd_list_append`](#cmd_list_append)


---
### cmd_list_print<!-- {{#callable:cmd_list_print}} -->
The `cmd_list_print` function generates a string representation of a list of commands, optionally escaping command separators.
- **Inputs**:
    - `cmdlist`: A pointer to a `cmd_list` structure, which contains a list of commands to be printed.
    - `escaped`: An integer flag indicating whether command separators should be escaped (non-zero value) or not (zero value).
- **Control Flow**:
    - Initialize a buffer `buf` with a length of 1 to store the resulting string.
    - Iterate over each command `cmd` in the `cmdlist` using `TAILQ_FOREACH`.
    - For each command, call [`cmd_print`](#cmd_print) to get its string representation and store it in `this`.
    - Increase the buffer size by the length of `this` plus 6 to accommodate separators and reallocate `buf` accordingly.
    - Concatenate `this` to `buf` using `strlcat`.
    - Determine the next command `next` in the list using `TAILQ_NEXT`.
    - If `next` is not NULL, check if the current command's group differs from the next command's group.
    - Depending on the `escaped` flag and group comparison, append either " ;; ", " \;\; ", " ; ", or " \; " to `buf`.
    - Free the memory allocated for `this`.
    - Return the final concatenated string `buf`.
- **Output**:
    - A dynamically allocated string containing the concatenated command representations, separated by appropriate delimiters.
- **Functions called**:
    - [`cmd_print`](#cmd_print)


---
### cmd_list_first<!-- {{#callable:cmd_list_first}} -->
The `cmd_list_first` function retrieves the first command from a given command list.
- **Inputs**:
    - `cmdlist`: A pointer to a `struct cmd_list`, which represents a list of commands.
- **Control Flow**:
    - The function calls `TAILQ_FIRST` with `cmdlist->list` to get the first element in the list.
    - It returns the result of `TAILQ_FIRST`, which is a pointer to the first `struct cmd` in the list.
- **Output**:
    - A pointer to the first `struct cmd` in the command list, or `NULL` if the list is empty.


---
### cmd_list_next<!-- {{#callable:cmd_list_next}} -->
The `cmd_list_next` function retrieves the next command in a linked list of commands.
- **Inputs**:
    - `cmd`: A pointer to the current command in the linked list.
- **Control Flow**:
    - The function uses the `TAILQ_NEXT` macro to access the next element in the linked list, using the `qentry` field of the `cmd` structure.
- **Output**:
    - A pointer to the next command in the linked list, or NULL if there is no next command.


---
### cmd_list_all_have<!-- {{#callable:cmd_list_all_have}} -->
The function `cmd_list_all_have` checks if all commands in a command list have a specific flag set.
- **Inputs**:
    - `cmdlist`: A pointer to a `struct cmd_list` which contains a list of commands to be checked.
    - `flag`: An integer representing the flag to be checked against each command's flags.
- **Control Flow**:
    - Iterate over each command in the command list using `TAILQ_FOREACH`.
    - For each command, check if the bitwise negation of the command's flags AND the given flag is non-zero.
    - If any command does not have the flag set, return 0 immediately.
    - If all commands have the flag set, return 1 after the loop completes.
- **Output**:
    - Returns 1 if all commands in the list have the specified flag set, otherwise returns 0.


---
### cmd_list_any_have<!-- {{#callable:cmd_list_any_have}} -->
The function `cmd_list_any_have` checks if any command in a command list has a specific flag set.
- **Inputs**:
    - `cmdlist`: A pointer to a `struct cmd_list`, which is a list of commands to be checked.
    - `flag`: An integer representing the flag to check against each command's flags.
- **Control Flow**:
    - Iterate over each command in the command list using `TAILQ_FOREACH` macro.
    - For each command, check if the command's flags include the specified flag using a bitwise AND operation.
    - If a command with the specified flag is found, return 1 immediately.
    - If no command with the specified flag is found after checking all commands, return 0.
- **Output**:
    - Returns 1 if any command in the list has the specified flag set, otherwise returns 0.


---
### cmd_mouse_at<!-- {{#callable:cmd_mouse_at}} -->
The `cmd_mouse_at` function calculates the mouse position relative to a given window pane and checks if it is within the pane's boundaries.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the window pane to check against.
    - `m`: A pointer to a `struct mouse_event` containing the mouse event data.
    - `xp`: A pointer to a `u_int` where the x-coordinate relative to the pane will be stored, if not NULL.
    - `yp`: A pointer to a `u_int` where the y-coordinate relative to the pane will be stored, if not NULL.
    - `last`: An integer flag indicating whether to use the last known mouse position (`1`) or the current position (`0`).
- **Control Flow**:
    - Determine the x and y coordinates based on the `last` flag, using either the last known or current mouse position combined with offsets.
    - Log the calculated x and y coordinates for debugging purposes.
    - Adjust the y-coordinate if the mouse event's status line is at the top and the y-coordinate is beyond the status lines.
    - Check if the calculated x and y coordinates are within the boundaries of the window pane; return -1 if they are not.
    - If the coordinates are within bounds, store the relative x and y coordinates in `xp` and `yp` if they are not NULL.
    - Return 0 to indicate success.
- **Output**:
    - Returns 0 if the mouse position is within the pane's boundaries, otherwise returns -1.


---
### cmd_mouse_window<!-- {{#callable:cmd_mouse_window}} -->
The `cmd_mouse_window` function retrieves the current window link associated with a mouse event in a session, if valid.
- **Inputs**:
    - `m`: A pointer to a `mouse_event` structure containing information about the mouse event, including its validity and associated session and window IDs.
    - `sp`: A pointer to a pointer to a `session` structure, which will be set to the session associated with the mouse event if it is valid and found.
- **Control Flow**:
    - Check if the mouse event `m` is valid; if not, return `NULL`.
    - Attempt to find the session `s` using the session ID from the mouse event `m`; if the session ID is invalid or the session is not found, return `NULL`.
    - If the window ID in the mouse event `m` is -1, set `wl` to the current window link of the session `s`.
    - If the window ID is not -1, attempt to find the window `w` using the window ID; if the window is not found, return `NULL`.
    - Find the window link `wl` in the session's window list that corresponds to the window `w`.
    - If the session pointer `sp` is not `NULL`, set it to the found session `s`.
    - Return the found window link `wl`.
- **Output**:
    - Returns a pointer to the `winlink` structure associated with the mouse event's window in the session, or `NULL` if the event is invalid or the session/window cannot be found.


---
### cmd_mouse_pane<!-- {{#callable:cmd_mouse_pane}} -->
The `cmd_mouse_pane` function retrieves the window pane associated with a mouse event, optionally updating session and winlink pointers.
- **Inputs**:
    - `m`: A pointer to a `mouse_event` structure containing details about the mouse event.
    - `sp`: A pointer to a pointer to a `session` structure, which may be updated to point to the session associated with the mouse event.
    - `wlp`: A pointer to a pointer to a `winlink` structure, which may be updated to point to the winlink associated with the mouse event.
- **Control Flow**:
    - Call [`cmd_mouse_window`](#cmd_mouse_window) with the mouse event `m` and session pointer `sp` to get the associated winlink `wl`.
    - If `wl` is NULL, return NULL indicating no associated window pane.
    - Check if the `wp` field in the mouse event is -1; if so, set `wp` to the active pane of the window in `wl`.
    - If `wp` is not -1, find the window pane by its ID using `window_pane_find_by_id`.
    - If the window pane is not found or is not part of the window in `wl`, return NULL.
    - If `wlp` is not NULL, update it to point to `wl`.
    - Return the window pane `wp`.
- **Output**:
    - Returns a pointer to the `window_pane` associated with the mouse event, or NULL if no such pane exists.
- **Functions called**:
    - [`cmd_mouse_window`](#cmd_mouse_window)


---
### cmd_template_replace<!-- {{#callable:cmd_template_replace}} -->
The `cmd_template_replace` function replaces the first occurrence of a placeholder in a template string with a given string, optionally escaping certain characters.
- **Inputs**:
    - `template`: A constant character pointer representing the template string containing placeholders to be replaced.
    - `s`: A constant character pointer representing the string to replace the placeholder with.
    - `idx`: An integer representing the index of the placeholder to be replaced, where placeholders are denoted by `%1`, `%2`, etc.
- **Control Flow**:
    - Check if the template contains any '%' character; if not, return a duplicate of the template string.
    - Initialize a buffer to build the result string and set initial conditions for replacement and length tracking.
    - Iterate over each character in the template string.
    - When a '%' character is encountered, check if the following character is a digit between '1' and '9' and matches the given index `idx`, or if it is another '%' character for literal replacement.
    - If a valid placeholder is found, replace it with the string `s`, escaping certain characters if necessary, and mark the replacement as done.
    - If no valid placeholder is found, append the current character to the buffer.
    - Continue processing until the end of the template string is reached.
    - Return the constructed buffer containing the modified template.
- **Output**:
    - A dynamically allocated string with the specified placeholder replaced by the given string, with certain characters escaped if necessary.


# Function Declarations (Public API)

---
### xvasprintf<!-- {{#callable_declaration:xvasprintf}} -->
Formats a string and stores it in a dynamically allocated buffer.
- **Description**: This function formats a string according to the specified format and argument list, storing the result in a dynamically allocated buffer. It is useful when you need to create a formatted string with variable arguments. The function must be called with a valid format string and a properly initialized va_list. It is important to check the return value to ensure the operation was successful, as a failure will result in program termination.
- **Inputs**:
    - `ret`: A pointer to a char pointer where the address of the allocated buffer will be stored. Must not be null. The caller is responsible for freeing the allocated memory.
    - `fmt`: A format string that specifies how to format the output. Must not be null.
    - `ap`: A va_list containing the arguments to format according to the format string. Must be properly initialized before calling this function.
- **Output**: Returns the number of characters printed (excluding the null byte used to end output to strings) on success. On failure, the program will terminate.
- **See also**: [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)  (Implementation)


