# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file primarily defines and manages command structures and operations within tmux. It includes a comprehensive list of command entries, each representing a specific command that tmux can execute, such as attaching a session, binding keys, managing panes, and handling buffers. These command entries are stored in an array, `cmd_table`, which serves as a registry for all available commands in tmux.

The file also provides a variety of utility functions for handling command arguments and command lists. These functions include operations for logging, appending, and packing argument vectors, as well as parsing and copying commands. Additionally, the file defines structures and functions for managing command lists, which are collections of commands that can be executed sequentially. The code also includes functions for handling mouse events within tmux, such as determining the current mouse position relative to a pane or window. Overall, this file is crucial for the command processing and execution logic in tmux, enabling users to interact with and control their terminal sessions effectively.
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
- **Description**: `cmd_attach_session_entry` is a global constant variable that represents a command entry structure for attaching to a session in the tmux terminal multiplexer. This structure likely contains function pointers and metadata related to the command's execution, such as its name, arguments, and usage information.
- **Use**: This variable is used to define the behavior and properties of the 'attach-session' command within the command processing system of tmux.


---
### cmd_bind_key_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_bind_key_entry` is a global constant variable that represents a command entry structure used in the command handling system of the application. It is part of a larger set of command entries that define various commands available to the user, specifically for binding keys to actions.
- **Use**: This variable is used to reference the command entry associated with binding keys in the command table.


---
### cmd_break_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_break_pane_entry` is a global constant variable that represents a command entry structure for breaking a pane in a terminal multiplexer. It is part of a larger command table that defines various commands available in the application, allowing for modular command handling.
- **Use**: This variable is used to reference the command entry associated with breaking a pane, enabling its functionality within the command processing system.


---
### cmd_capture_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_capture_pane_entry` is a global constant variable that represents a command entry structure for capturing the current pane in a terminal multiplexer application. It is part of a larger set of command entries that define various commands available in the application, each encapsulated in a `cmd_entry` structure.
- **Use**: This variable is used to reference the command entry associated with the 'capture pane' command in the command table.


---
### cmd_choose_buffer_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_choose_buffer_entry` is a global constant variable that represents a command entry structure for choosing a buffer in the application. It is part of a larger command table that defines various command entries used within the application, allowing for structured command handling.
- **Use**: This variable is used to reference the command entry associated with the buffer selection functionality in the command processing system.


---
### cmd_choose_client_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_choose_client_entry` variable is a global constant that represents a command entry structure for selecting a client in the command interface. It is part of a larger set of command entries that define various commands available in the application, allowing for structured command handling and execution.
- **Use**: This variable is used to reference the command entry associated with choosing a client, facilitating command execution within the application's command processing framework.


---
### cmd_choose_tree_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_choose_tree_entry` is a global constant variable that represents a command entry structure used within the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically related to choosing a tree structure.
- **Use**: This variable is used to reference the command entry for the 'choose tree' command in the command table.


---
### cmd_clear_history_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_clear_history_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a collection of command entries that define various commands available in the application, specifically for clearing the command history.
- **Use**: This variable is used to reference the command entry for clearing history in the command table.


---
### cmd_clear_prompt_history_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_clear_prompt_history_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically for clearing the prompt history.
- **Use**: This variable is used to reference the command entry associated with clearing the prompt history in the command processing logic.


---
### cmd_clock_mode_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_clock_mode_entry` is a global constant variable that represents a command entry structure specifically for the clock mode functionality within the application. This structure likely contains information about the command's name, its associated function, and any flags or parameters relevant to its execution.
- **Use**: This variable is used to define and register the clock mode command in the command table for the application.


---
### cmd_command_prompt_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_command_prompt_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically for handling command prompts.
- **Use**: This variable is used to reference the command entry associated with the command prompt functionality within the command processing framework.


---
### cmd_confirm_before_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_confirm_before_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a collection of command entries that define various commands available in the application, specifically related to confirming actions before they are executed.
- **Use**: This variable is used to reference the command entry for confirming actions before proceeding with their execution.


---
### cmd_copy_mode_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_copy_mode_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command entry for the copy mode functionality in the application. This structure likely contains information such as the command name, its associated function, and any flags or arguments related to the command.
- **Use**: This variable is used to provide access to the command entry for the copy mode, allowing the application to execute or reference this command when needed.


---
### cmd_customize_mode_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_customize_mode_entry` is a global constant variable that represents a command entry structure for customizing mode in the application. It is part of a larger command table that defines various commands available in the system, allowing for modular command handling.
- **Use**: This variable is used to reference the specific command entry for customizing mode within the command processing logic.


---
### cmd_delete_buffer_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_delete_buffer_entry` is a global constant variable that represents a command entry structure for the 'delete buffer' command in the application. It is part of a larger command table that defines various commands available in the system, allowing for structured command handling.
- **Use**: This variable is used to reference the specific command entry associated with deleting a buffer, facilitating command execution and management.


---
### cmd_detach_client_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_detach_client_entry` is a global constant variable that represents a command entry structure for detaching a client in the application. It is part of a command table that defines various commands available in the system, allowing for structured command handling.
- **Use**: This variable is used to reference the command entry associated with the detach client functionality within the command processing system.


---
### cmd_display_menu_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_display_menu_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a collection of command entries that define various commands available in the application, specifically related to displaying a menu.
- **Use**: This variable is used to reference the command entry associated with displaying a menu in the command table.


---
### cmd_display_message_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_display_message_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically for displaying messages to the user.
- **Use**: `cmd_display_message_entry` is utilized within the command table to facilitate the execution of the display message command when invoked by the user.


---
### cmd_display_popup_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_display_popup_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command for displaying a popup in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments relevant to its execution.
- **Use**: This variable is used to register and reference the command associated with displaying a popup in the command table.


---
### cmd_display_panes_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_display_panes_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to displaying panes in a terminal multiplexer context. This structure likely contains information such as the command name, its associated function, and any flags or arguments that pertain to the command's behavior.
- **Use**: This variable is used to register and access the command for displaying panes within the command handling system of the application.


---
### cmd_find_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_find_window_entry` is a global constant variable that represents a command entry structure for the 'find window' command in the application. It is part of a larger command table that defines various commands available in the system, allowing for structured command handling.
- **Use**: This variable is used to reference the specific command entry for the 'find window' command within the command processing logic.


---
### cmd_has_session_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_has_session_entry` is a global constant variable that represents a command entry structure for handling session-related commands in the application. It is part of a larger command table that defines various commands available in the system, allowing for structured command processing.
- **Use**: This variable is used to reference a specific command entry related to session management within the command processing framework.


---
### cmd_if_shell_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_if_shell_entry` is a global constant variable that represents a command entry structure for a specific command in the command table. It is part of a larger set of command entries that define various commands available in the application, likely related to shell operations.
- **Use**: This variable is used to reference the command entry for the 'if-shell' command within the command processing system.


---
### cmd_join_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_join_pane_entry` is a global constant variable that represents a command entry structure for the 'join pane' command in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: `cmd_join_pane_entry` is used to define the behavior and properties of the 'join pane' command within the command processing system of the application.


---
### cmd_kill_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_kill_pane_entry` is a global constant variable that represents a command entry structure specifically for the 'kill pane' command in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments that pertain to the command's execution.
- **Use**: This variable is used to define the behavior and properties of the 'kill pane' command within the command handling system.


---
### cmd_kill_server_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_kill_server_entry` is a global constant variable that represents a command entry structure specifically for the 'kill server' command in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments that pertain to its execution.
- **Use**: This variable is used to define the behavior and properties of the 'kill server' command within the command handling system.


---
### cmd_kill_session_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_kill_session_entry` is a global constant variable that represents a command entry structure specifically designed for the 'kill session' command in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments that pertain to the command's execution.
- **Use**: This variable is used to define the behavior and properties of the 'kill session' command within the command handling system.


---
### cmd_kill_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_kill_window_entry` is a global constant variable that represents a command entry structure specifically for the 'kill window' command in the application. This structure likely contains details such as the command name, its associated function, and any flags or arguments that pertain to the command's execution.
- **Use**: This variable is used to define the behavior and properties of the 'kill window' command within the command handling system.


---
### cmd_last_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_last_pane_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to the last pane in a terminal multiplexer context. This structure likely contains information such as the command's name, its associated function, and any flags or parameters relevant to its execution.
- **Use**: This variable is used to reference the command entry for the last pane, allowing the command system to access and execute the associated functionality.


---
### cmd_last_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_last_window_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to the last window in a terminal multiplexer context. This structure likely contains information such as the command's name, its associated function, and any flags or options relevant to its execution.
- **Use**: This variable is used to access the command entry for the last window, allowing the command processing system to invoke the appropriate behavior when the command is executed.


---
### cmd_link_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_link_window_entry` is a global constant variable that represents a command entry structure for linking a window in the command interface. It is part of a larger set of command entries that define various commands available in the application, specifically related to window management.
- **Use**: This variable is used to provide a reference to the command entry associated with linking a window, allowing the command system to recognize and execute the corresponding functionality.


---
### cmd_list_buffers_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_list_buffers_entry` is a global constant variable that represents a command entry structure specifically for listing buffers in the application. It is part of a larger set of command entries that define various commands available in the system, allowing for organized command handling and execution.
- **Use**: This variable is used to reference the command entry associated with the 'list buffers' command in the command table.


---
### cmd_list_clients_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_list_clients_entry` is a global variable that holds a reference to a `cmd_entry` structure, which defines a command related to listing clients in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: `cmd_list_clients_entry` is used to provide the command entry for listing clients, allowing the command parsing and execution system to recognize and handle this specific command.


---
### cmd_list_commands_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_list_commands_entry` is a global constant variable that represents a command entry structure, specifically designed to define a command related to listing commands in the application. This structure likely contains metadata and function pointers that facilitate the execution and handling of the command within the command processing framework.
- **Use**: This variable is used to reference the command entry for listing commands, allowing it to be included in command tables and invoked during command processing.


---
### cmd_list_keys_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_list_keys_entry` is a global variable that holds a constant reference to a `cmd_entry` structure, which defines a command related to listing key bindings in the application. This structure likely contains information such as the command name, its associated function, and any flags or arguments that pertain to the command's behavior.
- **Use**: `cmd_list_keys_entry` is used to provide the command entry for listing key bindings, allowing the application to recognize and execute this command when invoked by the user.


---
### cmd_list_panes_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_list_panes_entry` is a global constant variable that represents a command entry structure specifically for listing panes in a terminal multiplexer context. It is part of a larger set of command entries that define various commands available in the application.
- **Use**: This variable is used to provide the command entry details for the 'list panes' command within the command handling system.


---
### cmd_list_sessions_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_list_sessions_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to listing sessions in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to provide the command entry for listing sessions in the command processing system.


---
### cmd_list_windows_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_list_windows_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to listing windows in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to provide a command entry for listing windows, which can be referenced in command parsing and execution within the application.


---
### cmd_load_buffer_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_load_buffer_entry` variable is a global constant that represents a command entry structure for loading a buffer in the command processing system. It is part of a larger set of command entries that define various commands available in the application, each encapsulated in a `cmd_entry` structure.
- **Use**: This variable is used to reference the command entry associated with the load buffer command, allowing the command processing system to recognize and execute the corresponding functionality.


---
### cmd_lock_client_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_lock_client_entry` is a global constant variable that represents a command entry structure specifically for locking a client in the context of a command-line interface. This structure likely contains function pointers and metadata related to the command's execution, usage, and associated arguments.
- **Use**: This variable is used to define the behavior and properties of the 'lock client' command within the command processing system.


---
### cmd_lock_server_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_lock_server_entry` is a global constant variable that represents a command entry structure for locking the server in the context of a command processing system. It is part of a larger set of command entries that define various commands available in the application, each encapsulated in a `cmd_entry` structure.
- **Use**: This variable is used to reference the specific command entry for locking the server, allowing it to be included in command processing routines.


---
### cmd_lock_session_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_lock_session_entry` is a global constant variable that represents a command entry structure for locking a session in the application. It is part of a larger command table that defines various commands available in the system, allowing for structured command handling.
- **Use**: This variable is used to reference the command entry associated with locking a session, facilitating command execution and management within the application.


---
### cmd_move_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_move_pane_entry` is a global constant variable that represents a command entry structure specifically for the 'move pane' command in the application. This structure likely contains details such as the command's name, its associated arguments, and any flags or options relevant to its execution.
- **Use**: This variable is used to define the behavior and properties of the 'move pane' command within the command handling system.


---
### cmd_move_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_move_window_entry` is a global constant variable that represents a command entry structure for moving a window in the application. It is part of a larger command table that defines various commands available in the system, each associated with specific functionality.
- **Use**: This variable is used to reference the command entry for moving a window, allowing the application to execute the corresponding action when the command is invoked.


---
### cmd_new_session_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_new_session_entry` is a global constant variable that represents a command entry structure for creating a new session in the application. It is part of a command table that defines various commands available in the system, encapsulating the command's name, arguments, and associated functionality.
- **Use**: `cmd_new_session_entry` is used to reference the command entry for initiating a new session, allowing the command parsing and execution mechanisms to recognize and handle this specific command.


---
### cmd_new_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_new_window_entry` is a global constant variable that represents a command entry structure specifically for creating a new window in the application. It is part of a larger command table that defines various commands available in the system, each associated with specific functionalities.
- **Use**: This variable is used to reference the command entry for creating a new window, allowing the command parsing and execution mechanisms to recognize and handle this specific command.


---
### cmd_next_layout_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_next_layout_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to navigating to the next layout in a command-line interface. This structure likely contains information such as the command's name, its associated function, and any flags or parameters relevant to its execution.
- **Use**: This variable is used to facilitate the execution of the 'next layout' command within the command processing system.


---
### cmd_next_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_next_window_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command for navigating to the next window in a terminal multiplexer context. This structure likely contains details such as the command name, its associated function, and any flags or parameters relevant to its execution.
- **Use**: This variable is used to facilitate the execution of the command that allows users to switch to the next window in the command processing system.


---
### cmd_paste_buffer_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_paste_buffer_entry` variable is a global constant that represents a command entry for pasting from a buffer in the command system. It is part of a larger set of command entries that define various operations within the application, allowing users to interact with the command interface effectively.
- **Use**: This variable is used to reference the specific command entry associated with the paste buffer functionality in the command table.


---
### cmd_pipe_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: The variable `cmd_pipe_pane_entry` is a constant pointer to a `cmd_entry` structure, which defines a command related to piping data into a pane in a terminal multiplexer. This structure likely contains information such as the command name, its associated function, and any flags or options relevant to its execution.
- **Use**: This variable is used to reference the command entry for piping data into a pane, allowing it to be included in command tables or invoked as part of command processing.


---
### cmd_previous_layout_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_previous_layout_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to navigating to the previous layout in a command-line interface. This structure likely contains information such as the command name, its associated function, and any flags or parameters relevant to its execution.
- **Use**: `cmd_previous_layout_entry` is used to facilitate the execution of the command that allows users to revert to the previous layout in the application.


---
### cmd_previous_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_previous_window_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to navigating to the previous window in a terminal multiplexer context. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to facilitate the execution of the command that allows users to switch to the previous window in the command processing system.


---
### cmd_refresh_client_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_refresh_client_entry` is a global constant variable that represents a command entry structure for refreshing a client in the command processing system. It is part of a larger set of command entries that define various commands available in the application, each encapsulated in a `cmd_entry` structure.
- **Use**: This variable is used to reference the specific command entry for refreshing a client, allowing the command processing system to execute the associated functionality when invoked.


---
### cmd_rename_session_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_rename_session_entry` is a global constant variable that represents a command entry structure specifically for renaming a session in the application. It is part of a larger command table that defines various commands available in the system, allowing for structured command handling.
- **Use**: This variable is used to reference the command entry for renaming a session, facilitating command parsing and execution within the application.


---
### cmd_rename_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: The variable `cmd_rename_window_entry` is a constant pointer to a `cmd_entry` structure, which defines a command entry for renaming a window in the application. This structure likely contains information such as the command name, its associated function, and any flags or arguments related to the command.
- **Use**: This variable is used to reference the command entry for the 'rename window' command within the command table, allowing the application to execute the appropriate functionality when this command is invoked.


---
### cmd_resize_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_resize_pane_entry` is a global constant variable that represents a command entry structure specifically for resizing a pane in a terminal multiplexer application. It is part of a larger set of command entries that define various operations that can be performed within the application.
- **Use**: This variable is used to provide the command entry details for the resize pane operation, allowing it to be referenced in command parsing and execution.


---
### cmd_resize_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_resize_window_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines the properties and behavior of the 'resize window' command in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to register and provide access to the 'resize window' command within the command handling system of the application.


---
### cmd_respawn_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_respawn_pane_entry` is a global constant variable that represents a command entry structure for respawning a pane in the terminal multiplexer. It is part of a larger command table that defines various commands available in the application, allowing for structured command handling.
- **Use**: This variable is used to reference the command entry associated with the respawn pane command in the command processing logic.


---
### cmd_respawn_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_respawn_window_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to respawning a window in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to register and reference the respawn window command within the command handling system of the application.


---
### cmd_rotate_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_rotate_window_entry` is a global constant variable that represents a command entry structure for the 'rotate window' command in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to define the behavior and properties of the 'rotate window' command within the command handling system.


---
### cmd_run_shell_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_run_shell_entry` is a global constant variable that represents a command entry structure for executing shell commands within the application. It is part of a larger command table that defines various commands available to the user, encapsulating the command's properties and behavior.
- **Use**: This variable is used to reference the command entry associated with running shell commands, allowing the application to execute shell commands as part of its functionality.


---
### cmd_save_buffer_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_save_buffer_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically for saving a buffer.
- **Use**: This variable is used to reference the command entry associated with the 'save buffer' command in the command table.


---
### cmd_select_layout_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_select_layout_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to selecting a layout in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to provide a command entry for selecting a layout within the command processing system of the application.


---
### cmd_select_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_select_pane_entry` is a global constant variable that represents a command entry structure for selecting a pane in a terminal multiplexer application. It is part of a larger command table that defines various commands available to the user, encapsulating the command's name, arguments, and associated functionality.
- **Use**: This variable is used to provide the command entry details for the 'select pane' command within the command processing system.


---
### cmd_select_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: The variable `cmd_select_window_entry` is a constant pointer to a `cmd_entry` structure, which defines a command related to selecting a window in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to reference the command entry for selecting a window, allowing it to be included in command tables or invoked as part of command processing.


---
### cmd_send_keys_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_send_keys_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a collection of command entries that define various commands available in the application, specifically for sending key sequences.
- **Use**: This variable is used to reference the command entry associated with sending keys in the command processing logic.


---
### cmd_send_prefix_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_send_prefix_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically related to sending prefix commands.
- **Use**: This variable is used to reference the command entry for sending prefix commands within the command processing logic.


---
### cmd_server_access_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_server_access_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically related to server access functionalities.
- **Use**: This variable is used to reference the command entry for server access within the command table.


---
### cmd_set_buffer_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_set_buffer_entry` variable is a constant pointer to a `cmd_entry` structure, which defines a command related to setting a buffer in the application. This structure likely contains information such as the command's name, its function pointer, and any associated flags or arguments necessary for its execution.
- **Use**: This variable is used to reference the command entry for setting a buffer, allowing the command to be executed within the command processing framework.


---
### cmd_set_environment_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_set_environment_entry` is a global constant variable that represents a command entry structure specifically for setting environment variables within the application. It is part of a larger command table that defines various commands available in the system, allowing for structured command handling and execution.
- **Use**: This variable is used to reference the command entry associated with setting environment variables in the command processing logic.


---
### cmd_set_hook_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_set_hook_entry` is a global constant variable that represents a command entry structure for setting hooks in the command processing system. It is part of a larger command table that defines various commands available in the application, allowing for extensibility and customization of command behavior.
- **Use**: This variable is used to reference the command entry associated with setting hooks, enabling the command processing system to recognize and execute the corresponding functionality.


---
### cmd_set_option_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_set_option_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically for setting options.
- **Use**: This variable is used to reference the command entry for setting options within the command processing framework.


---
### cmd_set_window_option_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_set_window_option_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to setting window options in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to register and provide access to the command for setting window options within the command handling system.


---
### cmd_show_buffer_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_show_buffer_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to displaying buffer information in the application. This structure likely contains details such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to register and access the command for showing buffer entries within the command handling system.


---
### cmd_show_environment_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_show_environment_entry` is a global constant variable that holds a reference to a `cmd_entry` structure, which defines a command related to displaying environment variables in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to register and provide access to the command that shows environment entries within the command processing system.


---
### cmd_show_hooks_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_show_hooks_entry` is a global constant variable that represents a command entry structure for displaying hooks in the application. It is part of a larger command entry system that defines various commands available in the application, allowing for extensibility and modular command handling.
- **Use**: This variable is used to reference the command entry associated with showing hooks, enabling the application to execute the corresponding command when invoked.


---
### cmd_show_messages_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_show_messages_entry` is a global constant variable that represents a command entry structure for displaying messages in the application. It is part of a larger command table that defines various commands available in the system, allowing for structured command handling.
- **Use**: This variable is used to reference the command entry associated with showing messages, facilitating command execution and management.


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
- **Description**: `cmd_show_window_options_entry` is a global constant variable that represents a command entry structure for displaying window options in the application. It is part of a larger command table that defines various commands available in the system, allowing for structured command handling.
- **Use**: This variable is used to reference the command entry associated with showing window options, facilitating command execution and management.


---
### cmd_source_file_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_source_file_entry` is a global constant variable that represents a command entry structure, which is likely used to define a specific command in the command processing system. This structure is part of a larger command table that holds various command entries for a command-line interface, allowing for organized command management and execution.
- **Use**: This variable is used to reference a specific command entry within the command table, facilitating command execution and handling.


---
### cmd_split_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_split_window_entry` is a global constant variable that represents a command entry structure for splitting a window in the application. It is part of a larger command table that defines various commands available in the system, allowing for modular command handling.
- **Use**: This variable is used to reference the command entry associated with the split window functionality within the command processing system.


---
### cmd_start_server_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_start_server_entry` is a global constant variable that represents a command entry structure for starting a server in the application. It is part of a larger command table that defines various commands available in the system, each associated with specific functionalities.
- **Use**: This variable is used to reference the command entry for starting the server within the command processing system.


---
### cmd_suspend_client_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_suspend_client_entry` is a global constant variable that represents a command entry structure for suspending a client in the application. It is part of a larger command table that defines various commands available in the system, each associated with specific functionalities.
- **Use**: This variable is used to reference the command entry for suspending a client, allowing the command to be executed when invoked.


---
### cmd_swap_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_swap_pane_entry` is a global constant variable that represents a command entry structure for swapping panes in a terminal multiplexer application. It is part of a larger command table that defines various commands available to the user, encapsulating the command's properties such as its name, arguments, and associated functionality.
- **Use**: This variable is used to reference the command entry for swapping panes, allowing it to be accessed and executed within the command processing logic of the application.


---
### cmd_swap_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_swap_window_entry` is a global constant variable that represents a command entry structure for the 'swap window' command in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments it may require.
- **Use**: This variable is used to define the behavior and properties of the 'swap window' command within the command processing system.


---
### cmd_switch_client_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_switch_client_entry` is a global constant variable that represents a command entry structure used in the command processing system. It is part of a larger set of command entries that define various commands available in the application, specifically for switching between clients.
- **Use**: This variable is used to reference the command entry for switching clients within the command processing framework.


---
### cmd_unbind_key_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_unbind_key_entry` is a global constant variable that represents a command entry structure for unbinding a key in the command interface. It is part of a larger set of command entries that define various commands available in the application, allowing for extensibility and customization of key bindings.
- **Use**: This variable is used to reference the command entry associated with the unbinding of a key, facilitating command execution within the application.


---
### cmd_unlink_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_unlink_window_entry` is a global constant variable that represents a command entry structure for the 'unlink window' command in the application. This structure likely contains information such as the command's name, its associated function, and any flags or arguments that pertain to the command's execution.
- **Use**: This variable is used to define the behavior and properties of the 'unlink window' command within the command processing system.


---
### cmd_wait_for_entry
- **Type**: `const struct cmd_entry`
- **Description**: `cmd_wait_for_entry` is a global variable that holds a constant pointer to a `cmd_entry` structure, which defines a command that waits for user input. This structure likely contains information about the command's name, its function, and any associated arguments or flags.
- **Use**: `cmd_wait_for_entry` is used in the command table to facilitate the execution of the wait-for-entry command within the application.


---
### cmd_table
- **Type**: `const struct cmd_entry *cmd_table[]`
- **Description**: `cmd_table` is an array of pointers to `cmd_entry` structures, which represent various command entries in the application. Each entry in the array corresponds to a specific command that can be executed, providing a centralized way to manage and access command functionalities.
- **Use**: This variable is used to look up command entries by name, facilitating command execution and management within the application.


---
### cmd_list_next_group
- **Type**: `u_int`
- **Description**: `cmd_list_next_group` is a static global variable that holds the next group number for new command lists. It is initialized to 1 and is used to uniquely identify groups of commands within the command list structure.
- **Use**: This variable is incremented each time a new command list is created, ensuring that each list has a unique group identifier.


---
### i
- **Type**: `int`
- **Description**: The variable `i` is an integer used as an index to iterate over the elements of an argument vector (`argv`) within the `cmd_log_argv` function. It serves to access each argument in the vector during logging.
- **Use**: The variable `i` is used in a loop to log each argument in the `argv` array.


---
### argv
- **Type**: `char **`
- **Description**: `argv` is a pointer to an array of character pointers (strings) that represent the command-line arguments passed to the program. It is typically used in the context of the `main` function to access the arguments provided by the user when executing the program.
- **Use**: `argv` is used to log and manipulate command-line arguments within various functions.


# Data Structures

---
### cmd
- **Type**: `struct`
- **Members**:
    - `entry`: A pointer to a constant cmd_entry structure representing the command entry.
    - `args`: A pointer to an args structure holding the command's arguments.
    - `group`: An unsigned integer representing the group number of the command.
    - `file`: A pointer to a character string indicating the source file of the command.
    - `line`: An unsigned integer indicating the line number in the source file.
    - `qentry`: A TAILQ_ENTRY structure for linking commands in a queue.
- **Description**: The 'cmd' structure represents an instance of a command in the system, encapsulating details such as the command entry, its arguments, and metadata about its source location in the code. It is designed to be part of a linked list, allowing for the organization and execution of multiple commands in sequence. The structure is integral to managing command execution within the system, providing a way to store and access command-specific information efficiently.


# Functions

---
### cmd_prepend_argv<!-- {{#callable:cmd_prepend_argv}} -->
The `cmd_prepend_argv` function adds a new argument to the beginning of an existing argument vector, updating the argument count accordingly.
- **Inputs**:
    - `argc`: A pointer to an integer representing the number of arguments currently in the argument vector.
    - `argv`: A pointer to a pointer to a character array, representing the argument vector itself.
    - `arg`: A constant character pointer representing the new argument to be added to the beginning of the argument vector.
- **Control Flow**:
    - Allocate memory for a new argument vector that is one element larger than the current vector using [`xreallocarray`](xmalloc.c.driver.md#xreallocarray).
    - Copy the new argument `arg` into the first position of the new argument vector using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - Iterate over the existing argument vector and copy each element to the new vector, starting from the second position.
    - Free the memory allocated for the old argument vector using `free`.
    - Update the original argument vector pointer to point to the new vector.
    - Increment the argument count `argc` by one.
- **Output**:
    - The function does not return a value, but it modifies the input argument vector to include the new argument at the beginning and updates the argument count.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### cmd_append_argv<!-- {{#callable:cmd_append_argv}} -->
The `cmd_append_argv` function appends a new argument to an existing argument vector, reallocating memory as necessary.
- **Inputs**:
    - `argc`: A pointer to an integer representing the current number of arguments in the argument vector.
    - `argv`: A pointer to a pointer to a character array, representing the argument vector.
    - `arg`: A constant character pointer representing the new argument to be appended to the argument vector.
- **Control Flow**:
    - The function begins by reallocating memory for the argument vector to accommodate one additional argument using [`xreallocarray`](xmalloc.c.driver.md#xreallocarray).
    - The new argument is duplicated using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and assigned to the newly allocated space in the argument vector.
    - The argument count `argc` is incremented to reflect the addition of the new argument.
- **Output**:
    - The function does not return a value; it modifies the argument vector and its count in place.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### cmd_pack_argv<!-- {{#callable:cmd_pack_argv}} -->
The `cmd_pack_argv` function packs an argument vector into a buffer, ensuring each argument is null-terminated and fits within the specified buffer length.
- **Inputs**:
    - `argc`: The number of arguments in the argument vector.
    - `argv`: An array of strings representing the argument vector to be packed.
    - `buf`: A buffer where the packed argument vector will be stored.
    - `len`: The length of the buffer, indicating the maximum space available for packing the arguments.
- **Control Flow**:
    - If `argc` is zero, the function returns 0 immediately, indicating no arguments to pack.
    - The function logs the argument vector using `cmd_log_argv`.
    - The buffer is initialized to an empty string by setting the first character to '\0'.
    - A loop iterates over each argument in `argv`.
    - For each argument, [`strlcpy`](compat/strlcpy.c.driver.md#strlcpy) is used to copy the argument into the buffer, checking if the argument fits within the remaining buffer length.
    - If an argument does not fit, the function returns -1, indicating an error.
    - If the argument fits, the buffer pointer is advanced by the length of the copied argument plus one for the null terminator, and the remaining buffer length is decreased accordingly.
    - After all arguments are processed, the function returns 0, indicating success.
- **Output**:
    - The function returns 0 on success, indicating all arguments were packed into the buffer, or -1 if an argument could not fit within the buffer length.
- **Functions called**:
    - [`strlcpy`](compat/strlcpy.c.driver.md#strlcpy)


---
### cmd_unpack_argv<!-- {{#callable:cmd_unpack_argv}} -->
The `cmd_unpack_argv` function unpacks a buffer containing packed arguments into an argument vector.
- **Inputs**:
    - `buf`: A character buffer containing packed arguments, with each argument separated by a null character.
    - `len`: The length of the buffer `buf`.
    - `argc`: The number of arguments expected to be unpacked from the buffer.
    - `argv`: A pointer to a pointer to a character array, where the unpacked arguments will be stored.
- **Control Flow**:
    - Check if `argc` is zero; if so, return 0 immediately as there are no arguments to unpack.
    - Allocate memory for the argument vector `argv` using [`xcalloc`](xmalloc.c.driver.md#xcalloc), with space for `argc` pointers.
    - Set the last character of `buf` to a null terminator to ensure proper string termination.
    - Iterate over the number of arguments (`argc`), unpacking each argument from `buf`.
    - For each argument, check if `len` is zero, indicating insufficient buffer length; if so, free the allocated `argv` and return -1.
    - Calculate the length of the current argument using `strlen(buf) + 1` and duplicate the argument into `argv` using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - Advance the `buf` pointer and decrease `len` by the length of the current argument to move to the next argument.
    - Log the unpacked arguments using `cmd_log_argv`.
    - Return 0 to indicate successful unpacking.
- **Output**:
    - Returns 0 on success, or -1 if the buffer length is insufficient to unpack all arguments.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`cmd_free_argv`](#cmd_free_argv)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### cmd_copy_argv<!-- {{#callable:cmd_copy_argv}} -->
The `cmd_copy_argv` function duplicates an argument vector, ensuring it is terminated by a NULL pointer.
- **Inputs**:
    - `argc`: The number of arguments in the argument vector.
    - `argv`: The argument vector, which is an array of strings (char pointers).
- **Control Flow**:
    - Check if `argc` is zero; if so, return NULL immediately as there are no arguments to copy.
    - Allocate memory for a new argument vector `new_argv` with `argc + 1` elements, using [`xcalloc`](xmalloc.c.driver.md#xcalloc) to ensure it is zero-initialized.
    - Iterate over each element in the input `argv` array up to `argc`.
    - For each non-NULL element in `argv`, duplicate the string using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and assign it to the corresponding position in `new_argv`.
    - Return the newly created `new_argv` array.
- **Output**:
    - A newly allocated copy of the input argument vector, terminated by a NULL pointer, or NULL if `argc` is zero.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### cmd_free_argv<!-- {{#callable:cmd_free_argv}} -->
The `cmd_free_argv` function deallocates memory used by an argument vector.
- **Inputs**:
    - `argc`: The number of arguments in the argument vector.
    - `argv`: A pointer to an array of strings (argument vector) to be freed.
- **Control Flow**:
    - Check if `argc` is zero; if so, return immediately as there is nothing to free.
    - Iterate over each element in the `argv` array, freeing each string individually using `free`.
    - After all strings are freed, free the `argv` array itself.
- **Output**:
    - This function does not return any value; it performs memory deallocation.


---
### cmd_stringify_argv<!-- {{#callable:cmd_stringify_argv}} -->
The `cmd_stringify_argv` function converts an argument vector into a single space-separated string, escaping each argument as necessary.
- **Inputs**:
    - `argc`: The number of arguments in the argument vector.
    - `argv`: An array of strings representing the argument vector to be converted into a single string.
- **Control Flow**:
    - Check if `argc` is zero; if so, return an empty string using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - Initialize `buf` to `NULL` and `len` to zero to prepare for string concatenation.
    - Iterate over each argument in `argv` using a for loop.
    - For each argument, escape it using [`args_escape`](arguments.c.driver.md#args_escape) and store the result in `s`.
    - Log the escaped argument using `log_debug`.
    - Calculate the new length required for `buf` by adding the length of `s` plus one for a space or null terminator.
    - Reallocate `buf` to accommodate the new length using [`xrealloc`](xmalloc.c.driver.md#xrealloc).
    - If it's the first argument, initialize `buf` to an empty string; otherwise, append a space to `buf`.
    - Concatenate the escaped argument `s` to `buf` using [`strlcat`](compat/strlcat.c.driver.md#strlcat).
    - Free the memory allocated for `s` after it is used.
    - After processing all arguments, return the concatenated string `buf`.
- **Output**:
    - A dynamically allocated string containing the space-separated, escaped arguments from `argv`.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`args_escape`](arguments.c.driver.md#args_escape)
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)


---
### cmd_get_entry<!-- {{#callable:cmd_get_entry}} -->
The `cmd_get_entry` function retrieves the command entry associated with a given command structure.
- **Inputs**:
    - `cmd`: A pointer to a `struct cmd` which represents a command instance containing various attributes including the command entry.
- **Control Flow**:
    - The function takes a single argument, a pointer to a `struct cmd`.
    - It directly returns the `entry` member of the `cmd` structure, which is a pointer to a `struct cmd_entry`.
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
    - Returns a pointer to a `struct args`, which contains the arguments associated with the provided command.


---
### cmd_get_group<!-- {{#callable:cmd_get_group}} -->
The `cmd_get_group` function retrieves the group number associated with a given command structure.
- **Inputs**:
    - `cmd`: A pointer to a `struct cmd` which represents a command instance containing various attributes including the group number.
- **Control Flow**:
    - The function directly accesses the `group` member of the `cmd` structure and returns its value.
- **Output**:
    - The function returns an unsigned integer (`u_int`) representing the group number of the command.


---
### cmd_get_source<!-- {{#callable:cmd_get_source}} -->
The `cmd_get_source` function retrieves the source file name and line number associated with a command.
- **Inputs**:
    - `cmd`: A pointer to a `struct cmd` which contains information about a command, including its source file and line number.
    - `file`: A pointer to a `const char*` where the function will store the file name of the command's source, if not NULL.
    - `line`: A pointer to a `u_int` where the function will store the line number of the command's source, if not NULL.
- **Control Flow**:
    - Check if the `file` pointer is not NULL; if so, assign the `cmd->file` value to `*file`.
    - Check if the `line` pointer is not NULL; if so, assign the `cmd->line` value to `*line`.
- **Output**:
    - The function does not return a value, but it outputs the file name and line number of the command's source through the provided pointers.


---
### cmd_get_alias<!-- {{#callable:cmd_get_alias}} -->
The `cmd_get_alias` function retrieves the alias for a given command name from the global options if it exists.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the command for which the alias is being searched.
- **Control Flow**:
    - Retrieve the 'command-alias' options entry from the global options.
    - If the options entry is NULL, return NULL indicating no alias is found.
    - Calculate the length of the input command name.
    - Iterate over each item in the options array associated with the 'command-alias'.
    - For each item, retrieve its value and locate the '=' character in the string.
    - If '=' is found, calculate the length of the substring before '=' and compare it with the length of the input command name.
    - If the lengths match and the substrings are equal, duplicate the string after '=' and return it as the alias.
    - If no matching alias is found after iterating through all items, return NULL.
- **Output**:
    - Returns a duplicated string of the alias if found, otherwise returns NULL.
- **Functions called**:
    - [`options_get_only`](options.c.driver.md#options_get_only)
    - [`options_array_first`](options.c.driver.md#options_array_first)
    - [`options_array_item_value`](options.c.driver.md#options_array_item_value)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`options_array_next`](options.c.driver.md#options_array_next)


---
### cmd_find<!-- {{#callable:cmd_find}} -->
The `cmd_find` function searches for a command entry by its name in a predefined command table and handles cases of ambiguous or unknown commands.
- **Inputs**:
    - `name`: A string representing the name of the command to be searched for.
    - `cause`: A pointer to a string where an error message will be stored if the command is ambiguous or unknown.
- **Control Flow**:
    - Initialize `ambiguous` to 0 and `found` to NULL.
    - Iterate over the `cmd_table` array to find a matching command entry.
    - If an entry's alias matches the `name`, set `found` to this entry and break the loop.
    - If an entry's name partially matches the `name`, check if `found` is already set to mark as ambiguous, otherwise set `found` to this entry.
    - If an entry's name exactly matches the `name`, break the loop.
    - If `ambiguous` is set, construct an error message listing possible matches and return NULL.
    - If no entry is found, construct an error message indicating the command is unknown and return NULL.
    - Return the found command entry if no errors occurred.
- **Output**:
    - Returns a pointer to the `cmd_entry` structure if a unique match is found, otherwise returns NULL and sets an error message in `cause`.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)


---
### cmd_parse<!-- {{#callable:cmd_parse}} -->
The `cmd_parse` function parses a command from an argument vector, validates it, and constructs a command structure if successful.
- **Inputs**:
    - `values`: An array of `args_value` structures representing the command arguments.
    - `count`: The number of elements in the `values` array.
    - `file`: A string representing the file name where the command is defined, or NULL if not applicable.
    - `line`: An unsigned integer representing the line number in the file where the command is defined.
    - `cause`: A pointer to a string where error messages can be stored if the function fails.
- **Control Flow**:
    - Check if `count` is zero or if the first element of `values` is not of type `ARGS_STRING`; if so, set `cause` to "no command" and return NULL.
    - Use [`cmd_find`](#cmd_find) to locate the command entry corresponding to the first string in `values`; if not found, return NULL.
    - Parse the arguments using [`args_parse`](tmux.h.driver.md#args_parse) with the command entry's argument specifications; if parsing fails and no error message is set, set `cause` to the command's usage and return NULL.
    - If parsing fails with an error message, set `cause` to the error message, free the error string, and return NULL.
    - Allocate memory for a new `cmd` structure, set its `entry` and `args` fields, and copy the `file` and `line` information if provided.
    - Return the constructed `cmd` structure.
- **Output**:
    - Returns a pointer to a `cmd` structure if successful, or NULL if an error occurs, with `cause` set to an appropriate error message.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`cmd_find`](#cmd_find)
    - [`args_parse`](tmux.h.driver.md#args_parse)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### cmd_free<!-- {{#callable:cmd_free}} -->
The `cmd_free` function deallocates memory associated with a `cmd` structure, including its file and arguments.
- **Inputs**:
    - `cmd`: A pointer to a `struct cmd` that represents a command instance, which includes a file, arguments, and other metadata.
- **Control Flow**:
    - The function first frees the memory allocated for the `file` member of the `cmd` structure using `free(cmd->file)`.
    - It then calls `args_free(cmd->args)` to deallocate the memory associated with the command's arguments.
    - Finally, it frees the memory allocated for the `cmd` structure itself using `free(cmd)`.
- **Output**:
    - The function does not return any value; it performs memory deallocation for the given `cmd` structure and its components.
- **Functions called**:
    - [`args_free`](arguments.c.driver.md#args_free)


---
### cmd_copy<!-- {{#callable:cmd_copy}} -->
The `cmd_copy` function creates a duplicate of a given command structure, optionally replacing its arguments with new ones.
- **Inputs**:
    - `cmd`: A pointer to the command structure (`struct cmd`) that is to be copied.
    - `argc`: An integer representing the number of arguments in the new argument vector.
    - `argv`: A pointer to an array of strings representing the new argument vector to be used in the copied command.
- **Control Flow**:
    - Allocate memory for a new command structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc) and assign it to `new_cmd`.
    - Copy the `entry` field from the original `cmd` to `new_cmd`.
    - Copy the arguments from the original `cmd` using [`args_copy`](arguments.c.driver.md#args_copy), passing `argc` and `argv` to potentially replace the arguments, and assign the result to `new_cmd->args`.
    - If the `file` field in the original `cmd` is not NULL, duplicate the string using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and assign it to `new_cmd->file`.
    - Copy the `line` field from the original `cmd` to `new_cmd`.
    - Return the newly created `new_cmd` structure.
- **Output**:
    - Returns a pointer to a newly allocated `struct cmd` that is a copy of the input command, with potentially new arguments.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`args_copy`](arguments.c.driver.md#args_copy)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### cmd_print<!-- {{#callable:cmd_print}} -->
The `cmd_print` function generates a string representation of a command by combining its name and arguments.
- **Inputs**:
    - `cmd`: A pointer to a `struct cmd` which contains the command entry and its arguments.
- **Control Flow**:
    - Call [`args_print`](arguments.c.driver.md#args_print) to convert the command's arguments into a string `s`.
    - Check if the string `s` is not empty; if true, format the output string `out` to include both the command name and arguments.
    - If `s` is empty, duplicate the command name into `out`.
    - Free the memory allocated for `s`.
- **Output**:
    - Returns a dynamically allocated string `out` that represents the command, including its name and arguments if any.
- **Functions called**:
    - [`args_print`](arguments.c.driver.md#args_print)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### cmd_list_new<!-- {{#callable:cmd_list_new}} -->
The `cmd_list_new` function creates and initializes a new command list structure.
- **Inputs**:
    - None
- **Control Flow**:
    - Allocate memory for a new `cmd_list` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc) and assign it to `cmdlist`.
    - Set the `references` field of `cmdlist` to 1.
    - Assign a unique group number to `cmdlist->group` by incrementing `cmd_list_next_group`.
    - Allocate memory for the `list` field of `cmdlist` using [`xcalloc`](xmalloc.c.driver.md#xcalloc).
    - Initialize the `list` field as an empty tail queue using `TAILQ_INIT`.
    - Return the pointer to the newly created `cmd_list` structure.
- **Output**:
    - A pointer to the newly created and initialized `cmd_list` structure.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### cmd_list_append<!-- {{#callable:cmd_list_append}} -->
The `cmd_list_append` function appends a command to a command list, setting the command's group to match the list's group.
- **Inputs**:
    - `cmdlist`: A pointer to a `struct cmd_list`, representing the list to which the command will be appended.
    - `cmd`: A pointer to a `struct cmd`, representing the command to be appended to the list.
- **Control Flow**:
    - The function sets the `group` field of the `cmd` structure to the `group` of the `cmdlist` structure.
    - It then inserts the `cmd` at the tail of the `cmdlist->list` using the `TAILQ_INSERT_TAIL` macro.
- **Output**:
    - The function does not return a value; it modifies the `cmdlist` and `cmd` structures in place.


---
### cmd_list_append_all<!-- {{#callable:cmd_list_append_all}} -->
The `cmd_list_append_all` function appends all commands from one command list to another, updating the group of each command to match the destination list.
- **Inputs**:
    - `cmdlist`: A pointer to the destination `cmd_list` where commands will be appended.
    - `from`: A pointer to the source `cmd_list` from which commands will be taken.
- **Control Flow**:
    - Iterate over each command in the source list `from` using `TAILQ_FOREACH`.
    - For each command, set its `group` attribute to match the `group` of the destination list `cmdlist`.
    - Concatenate the source list `from->list` to the destination list `cmdlist->list` using `TAILQ_CONCAT`.
- **Output**:
    - The function does not return a value; it modifies the destination `cmd_list` in place by appending commands from the source list.


---
### cmd_list_move<!-- {{#callable:cmd_list_move}} -->
The `cmd_list_move` function transfers all commands from one command list to another and assigns a new group number to the destination list.
- **Inputs**:
    - `cmdlist`: A pointer to the destination command list where commands will be moved.
    - `from`: A pointer to the source command list from which commands will be moved.
- **Control Flow**:
    - Concatenate the list of commands from the 'from' command list to the 'cmdlist' command list using the TAILQ_CONCAT macro.
    - Assign a new group number to the 'cmdlist' by incrementing the global variable 'cmd_list_next_group'.
- **Output**:
    - The function does not return a value; it modifies the 'cmdlist' in place by moving commands from 'from' and updating its group number.


---
### cmd_list_free<!-- {{#callable:cmd_list_free}} -->
The `cmd_list_free` function deallocates a command list and its associated resources if it is no longer referenced.
- **Inputs**:
    - `cmdlist`: A pointer to a `struct cmd_list` which represents the command list to be freed.
- **Control Flow**:
    - Decrement the reference count of the command list.
    - If the reference count is not zero, return immediately without freeing resources.
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
    - `argv`: An array of strings (`char **`) containing arguments to be used for expanding placeholders in the command arguments.
- **Control Flow**:
    - The function begins by printing the original command list for debugging purposes.
    - A new command list is created using [`cmd_list_new`](#cmd_list_new).
    - The function iterates over each command in the original command list using `TAILQ_FOREACH`.
    - For each command, it checks if the command's group is different from the current group; if so, it updates the new command list's group and the current group variable.
    - Each command is copied using [`cmd_copy`](#cmd_copy), with the `argc` and `argv` arguments used to expand placeholders in the command's arguments.
    - The copied command is appended to the new command list using [`cmd_list_append`](#cmd_list_append).
    - After processing all commands, the new command list is printed for debugging purposes.
    - Finally, the new command list is returned.
- **Output**:
    - The function returns a pointer to the newly created command list (`struct cmd_list *`) that is a copy of the original, with placeholders in arguments expanded.
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
    - Initialize a buffer with a length of 1 to store the resulting string.
    - Iterate over each command in the `cmdlist` using `TAILQ_FOREACH`.
    - For each command, convert it to a string using [`cmd_print`](#cmd_print) and append it to the buffer, reallocating the buffer as needed to accommodate the new string.
    - Check if there is a next command in the list using `TAILQ_NEXT`.
    - If there is a next command, determine if the current and next commands belong to different groups.
    - If they belong to different groups, append either ' ;; ' or ' \;\; ' to the buffer based on the `escaped` flag.
    - If they belong to the same group, append either ' ; ' or ' \; ' to the buffer based on the `escaped` flag.
    - Free the temporary string representation of the current command.
    - Return the final buffer containing the concatenated string representation of the command list.
- **Output**:
    - A dynamically allocated string containing the concatenated and formatted representation of the command list, with separators based on group membership and the `escaped` flag.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`cmd_print`](#cmd_print)
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)


---
### cmd_list_first<!-- {{#callable:cmd_list_first}} -->
The `cmd_list_first` function retrieves the first command from a given command list.
- **Inputs**:
    - `cmdlist`: A pointer to a `struct cmd_list`, which represents a list of commands.
- **Control Flow**:
    - The function calls `TAILQ_FIRST` macro with `cmdlist->list` as an argument to get the first element of the list.
- **Output**:
    - Returns a pointer to the first `struct cmd` in the command list, or NULL if the list is empty.


---
### cmd_list_next<!-- {{#callable:cmd_list_next}} -->
The `cmd_list_next` function retrieves the next command in a linked list of commands.
- **Inputs**:
    - `cmd`: A pointer to the current command in the linked list from which the next command is to be retrieved.
- **Control Flow**:
    - The function uses the `TAILQ_NEXT` macro to access the next element in the linked list, using the `qentry` field of the `cmd` structure to traverse the list.
- **Output**:
    - Returns a pointer to the next `cmd` structure in the linked list, or `NULL` if there is no next element.


---
### cmd_list_all_have<!-- {{#callable:cmd_list_all_have}} -->
The function `cmd_list_all_have` checks if all commands in a command list have a specific flag set.
- **Inputs**:
    - `cmdlist`: A pointer to a `struct cmd_list` which contains a list of commands to be checked.
    - `flag`: An integer representing the flag to be checked against each command's flags.
- **Control Flow**:
    - Iterate over each command in the command list using `TAILQ_FOREACH`.
    - For each command, check if the bitwise negation of the command's flags AND the input flag is non-zero.
    - If any command does not have the flag set, return 0 immediately.
    - If all commands have the flag set, return 1 after the loop completes.
- **Output**:
    - Returns 1 if all commands in the list have the specified flag set, otherwise returns 0.


---
### cmd_list_any_have<!-- {{#callable:cmd_list_any_have}} -->
The function `cmd_list_any_have` checks if any command in a given command list has a specified flag set.
- **Inputs**:
    - `cmdlist`: A pointer to a `struct cmd_list` which contains a list of commands to be checked.
    - `flag`: An integer representing the flag to be checked against each command's flags.
- **Control Flow**:
    - Iterate over each command in the command list using the `TAILQ_FOREACH` macro.
    - For each command, check if the command's flags include the specified flag using a bitwise AND operation.
    - If a command with the specified flag is found, return 1 immediately.
    - If no command with the specified flag is found after checking all commands, return 0.
- **Output**:
    - Returns 1 if any command in the list has the specified flag set, otherwise returns 0.


---
### cmd_mouse_at<!-- {{#callable:cmd_mouse_at}} -->
The `cmd_mouse_at` function calculates the mouse position relative to a given window pane and checks if it is within the pane's boundaries.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure representing the window pane to check against.
    - `m`: A pointer to a `mouse_event` structure containing the mouse event data.
    - `xp`: A pointer to an unsigned integer where the x-coordinate relative to the pane will be stored, if not NULL.
    - `yp`: A pointer to an unsigned integer where the y-coordinate relative to the pane will be stored, if not NULL.
    - `last`: An integer flag indicating whether to use the last known mouse position (`lx`, `ly`) or the current position (`x`, `y`).
- **Control Flow**:
    - Determine the x and y coordinates based on the `last` flag, using either the last known or current mouse position combined with offsets.
    - Log the calculated x and y coordinates for debugging purposes.
    - Adjust the y-coordinate if the mouse event's status line is at the top and the y-coordinate is beyond the status lines.
    - Check if the x-coordinate is outside the horizontal bounds of the window pane; return -1 if it is.
    - Check if the y-coordinate is outside the vertical bounds of the window pane; return -1 if it is.
    - If `xp` is not NULL, store the x-coordinate relative to the pane in `*xp`.
    - If `yp` is not NULL, store the y-coordinate relative to the pane in `*yp`.
    - Return 0 to indicate the mouse position is within the pane's boundaries.
- **Output**:
    - Returns 0 if the mouse position is within the pane's boundaries, otherwise returns -1.


---
### cmd_mouse_window<!-- {{#callable:cmd_mouse_window}} -->
The `cmd_mouse_window` function retrieves the current window link associated with a mouse event in a session, if valid.
- **Inputs**:
    - `m`: A pointer to a `mouse_event` structure that contains information about the mouse event, including its validity and associated session and window IDs.
    - `sp`: A double pointer to a `session` structure, which will be set to the session associated with the mouse event if it is valid and found.
- **Control Flow**:
    - Check if the mouse event `m` is valid; if not, return `NULL`.
    - Attempt to find the session using the session ID from the mouse event `m`; if the session ID is invalid or the session is not found, return `NULL`.
    - If the window ID in the mouse event `m` is -1, set the window link `wl` to the current window of the session `s`.
    - If the window ID is not -1, attempt to find the window using the window ID; if not found, return `NULL`.
    - Find the window link `wl` in the session's window list using the found window `w`.
    - If the session pointer `sp` is not `NULL`, set it to the found session `s`.
    - Return the found window link `wl`.
- **Output**:
    - Returns a pointer to the `winlink` structure associated with the mouse event's window in the session, or `NULL` if the event is invalid or the session/window cannot be found.
- **Functions called**:
    - [`session_find_by_id`](session.c.driver.md#session_find_by_id)
    - [`window_find_by_id`](window.c.driver.md#window_find_by_id)
    - [`winlink_find_by_window`](window.c.driver.md#winlink_find_by_window)


---
### cmd_mouse_pane<!-- {{#callable:cmd_mouse_pane}} -->
The `cmd_mouse_pane` function retrieves the window pane associated with a mouse event, updating session and winlink pointers if provided.
- **Inputs**:
    - `m`: A pointer to a `mouse_event` structure that contains information about the mouse event.
    - `sp`: A pointer to a pointer to a `session` structure, which will be updated to point to the session associated with the mouse event if not NULL.
    - `wlp`: A pointer to a pointer to a `winlink` structure, which will be updated to point to the winlink associated with the mouse event if not NULL.
- **Control Flow**:
    - Call [`cmd_mouse_window`](#cmd_mouse_window) with the mouse event `m` and session pointer `sp` to get the associated winlink `wl`.
    - If `wl` is NULL, return NULL indicating no associated window pane.
    - Check if the `wp` field in the mouse event is -1; if so, set `wp` to the active pane of the window in `wl`.
    - If `wp` is not -1, find the window pane by its ID using [`window_pane_find_by_id`](window.c.driver.md#window_pane_find_by_id).
    - If the window pane is not found or does not belong to the window in `wl`, return NULL.
    - If `wlp` is not NULL, update it to point to `wl`.
    - Return the window pane `wp`.
- **Output**:
    - Returns a pointer to the `window_pane` associated with the mouse event, or NULL if no such pane exists.
- **Functions called**:
    - [`cmd_mouse_window`](#cmd_mouse_window)
    - [`window_pane_find_by_id`](window.c.driver.md#window_pane_find_by_id)
    - [`window_has_pane`](window.c.driver.md#window_has_pane)


---
### cmd_template_replace<!-- {{#callable:cmd_template_replace}} -->
The function `cmd_template_replace` replaces the first occurrence of a placeholder in a template string with a given string, optionally escaping certain characters.
- **Inputs**:
    - `template`: A constant character pointer representing the template string containing placeholders to be replaced.
    - `s`: A constant character pointer representing the string to replace the placeholder with.
    - `idx`: An integer representing the index of the placeholder to be replaced, where placeholders are denoted by '%1' to '%9'.
- **Control Flow**:
    - Check if the template contains any '%' character; if not, return a duplicate of the template string.
    - Initialize a buffer to store the result and set initial conditions for replacement and length tracking.
    - Iterate over each character in the template string.
    - When a '%' character is encountered, check if the following character is a digit between '1' and '9' and matches the given index, or if it is another '%' character for literal replacement.
    - If a valid placeholder is found, replace it with the string `s`, escaping certain characters if necessary, and update the buffer.
    - If no valid placeholder is found, append the current character to the buffer.
    - Continue until the end of the template string is reached.
    - Return the modified buffer.
- **Output**:
    - A dynamically allocated string with the specified placeholder replaced by the given string, with certain characters escaped if necessary.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)


