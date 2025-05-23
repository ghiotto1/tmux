# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file defines a comprehensive set of configuration options for tmux, including server, session, and window options. These options are organized into tables that specify their types, default values, and constraints, such as range limits and valid choices. The file includes various option types, such as numbers, strings, colors, and choices, and it provides detailed descriptions for each option, explaining their purpose and usage within tmux.

The file also defines several lists of valid choices for specific options, such as cursor styles, status line positions, and pane border indicators. Additionally, it includes macros for defining hook options, which are used to execute commands in response to specific events within tmux, such as window resizing or session creation. The file serves as a central repository for tmux's configuration options, ensuring that all options are consistently defined and documented. It is intended to be included in other parts of the tmux codebase, where these options are accessed and manipulated to control tmux's behavior and appearance.
# Imports and Dependencies

---
- `sys/types.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### options_table_mode_keys_list
- **Type**: ``const char *` array`
- **Description**: The `options_table_mode_keys_list` is a static constant array of strings that contains the available key mode options for a software configuration. It includes the options "emacs" and "vi", which are common keybinding styles used in text editors and command-line interfaces. The array is terminated with a `NULL` pointer to indicate the end of the list.
- **Use**: This variable is used to define the available choices for key mode settings in the software's configuration options.


---
### options_table_clock_mode_style_list
- **Type**: `const char *[]`
- **Description**: The `options_table_clock_mode_style_list` is a static constant array of string literals representing the available clock mode styles. It contains two options: "12" and "24", which correspond to 12-hour and 24-hour clock formats, respectively.
- **Use**: This variable is used to define the available choices for the clock mode style option in the application, allowing users to select between 12-hour and 24-hour time formats.


---
### options_table_status_list
- **Type**: `const char *[]`
- **Description**: The `options_table_status_list` is a static constant array of strings that represents different status options available in the application. The array includes the strings "off", "on", "2", "3", "4", "5", and ends with a NULL pointer to indicate the end of the list.
- **Use**: This variable is used to define the possible values for a status option in the application's configuration or settings.


---
### options_table_message_line_list
- **Type**: `const char *[]`
- **Description**: The `options_table_message_line_list` is a static constant array of string literals representing different message line options. It contains the strings "0", "1", "2", "3", "4", and a terminating NULL pointer.
- **Use**: This array is used to define the available choices for the 'message-line' option in the session configuration, allowing users to select the position of messages and the command prompt.


---
### options_table_status_keys_list
- **Type**: `const char *[]`
- **Description**: The `options_table_status_keys_list` is a static constant array of strings that contains the key set options for the status line in the tmux configuration. It includes two options: "emacs" and "vi", which are common keybinding styles used in text editors.
- **Use**: This variable is used to define the available keybinding styles for the status line in tmux, allowing users to choose between emacs and vi key sets.


---
### options_table_status_justify_list
- **Type**: ``const char *[]``
- **Description**: The `options_table_status_justify_list` is a static constant array of strings that contains options for justifying the status line in a user interface. The options include "left", "centre", "right", and "absolute-centre".
- **Use**: This variable is used to provide a list of possible justification options for the status line in the application, allowing users to select how the status line should be aligned.


---
### options_table_status_position_list
- **Type**: `const char *[]`
- **Description**: The `options_table_status_position_list` is a static constant array of strings that contains the possible values for the position of the status line in a user interface. The array includes two options: "top" and "bottom", followed by a NULL terminator to indicate the end of the list.
- **Use**: This variable is used to define the available choices for setting the position of the status line in the session options.


---
### options_table_bell_action_list
- **Type**: ``const char *``
- **Description**: The `options_table_bell_action_list` is a static constant array of strings that defines the possible actions that can be taken in response to a bell alert in the tmux application. The array contains the options "none", "any", "current", and "other", with a terminating NULL pointer to indicate the end of the list.
- **Use**: This variable is used to provide a list of choices for configuring the action to take when a bell alert occurs in a tmux session.


---
### options_table_visual_bell_list
- **Type**: `const char *[]`
- **Description**: The `options_table_visual_bell_list` is a static constant array of strings that represents the possible settings for visual bell alerts in the application. It includes options such as 'off', 'on', and 'both', with a terminating NULL to indicate the end of the list.
- **Use**: This variable is used to define the available choices for configuring how visual bell alerts are displayed in the application.


---
### options_table_cursor_style_list
- **Type**: `const char *[]`
- **Description**: The `options_table_cursor_style_list` is a static constant array of strings that defines a list of possible cursor styles. It includes styles such as 'default', 'blinking-block', 'block', 'blinking-underline', 'underline', 'blinking-bar', and 'bar'. The array is terminated with a NULL pointer to indicate the end of the list.
- **Use**: This array is used to provide a set of predefined cursor style options that can be selected by the user or application.


---
### options_table_pane_scrollbars_list
- **Type**: `const char *[]`
- **Description**: The `options_table_pane_scrollbars_list` is a static constant array of strings that defines the possible states for pane scrollbars in the application. The array includes the options "off", "modal", and "on", with a terminating NULL pointer to indicate the end of the list.
- **Use**: This variable is used to provide a list of valid choices for configuring the state of pane scrollbars in the application.


---
### options_table_pane_scrollbars_position_list
- **Type**: `const char *[]`
- **Description**: The `options_table_pane_scrollbars_position_list` is a static constant array of strings that specifies the possible positions for pane scrollbars in a user interface. It contains two options: "right" and "left", followed by a NULL terminator to indicate the end of the list.
- **Use**: This variable is used to define the available choices for the position of pane scrollbars in the configuration of a software application, likely a terminal multiplexer like tmux.


---
### options_table_pane_status_list
- **Type**: `const char *[]`
- **Description**: The `options_table_pane_status_list` is a static constant array of strings that defines the possible status positions for a pane in the tmux application. The array contains three string elements: "off", "top", and "bottom", followed by a NULL terminator to indicate the end of the list.
- **Use**: This variable is used to provide a list of valid options for setting the position of the pane status line in tmux.


---
### options_table_pane_border_indicators_list
- **Type**: `const char *[]`
- **Description**: The `options_table_pane_border_indicators_list` is a static constant array of strings that defines the possible options for pane border indicators in a terminal multiplexer environment. The options include "off", "colour", "arrows", and "both", with a terminating NULL to indicate the end of the list.
- **Use**: This variable is used to provide a list of choices for configuring how active pane borders are indicated, either by color, arrows, both, or not at all.


---
### options_table_pane_border_lines_list
- **Type**: `const char *[]`
- **Description**: The `options_table_pane_border_lines_list` is a static constant array of strings that defines the different styles of border lines that can be used for pane borders in the application. The available styles include "single", "double", "heavy", "simple", and "number". This array is terminated with a NULL pointer to indicate the end of the list.
- **Use**: This variable is used to provide a list of available pane border line styles for configuration or selection within the application.


---
### options_table_popup_border_lines_list
- **Type**: ``const char *``
- **Description**: The `options_table_popup_border_lines_list` is a static constant array of strings that defines the different styles of border lines that can be used for popups. The array includes styles such as "single", "double", "heavy", "simple", "rounded", "padded", and "none". The array is terminated with a `NULL` pointer to indicate the end of the list.
- **Use**: This variable is used to provide a list of available border line styles for configuring the appearance of popups in the application.


---
### options_table_set_clipboard_list
- **Type**: `const char *[]`
- **Description**: The `options_table_set_clipboard_list` is a static constant array of strings that defines the possible values for the clipboard setting in the tmux configuration. The array includes the options "off", "external", and "on", with a terminating NULL pointer to indicate the end of the list.
- **Use**: This variable is used to provide a list of valid choices for the 'set-clipboard' option in tmux, allowing users to configure how tmux interacts with the system clipboard.


---
### options_table_window_size_list
- **Type**: `const char *[]`
- **Description**: The `options_table_window_size_list` is a static constant array of strings that defines the possible values for window size options in the tmux configuration. It includes options such as 'largest', 'smallest', 'manual', and 'latest', with a terminating NULL pointer to indicate the end of the list.
- **Use**: This variable is used to provide a predefined set of choices for configuring how window sizes are determined in tmux.


---
### options_table_remain_on_exit_list
- **Type**: ``const char *` array`
- **Description**: The `options_table_remain_on_exit_list` is a static constant array of strings that represents the possible values for the 'remain-on-exit' option in the tmux configuration. This array includes the options 'off', 'on', and 'failed', followed by a NULL terminator to indicate the end of the list.
- **Use**: This variable is used to define the valid choices for the 'remain-on-exit' option, which determines whether panes should remain open or be automatically closed when the program inside exits.


---
### options_table_destroy_unattached_list
- **Type**: `const char *[]`
- **Description**: The `options_table_destroy_unattached_list` is a static constant array of strings that defines the possible options for handling sessions that have no attached clients in the tmux application. The options include "off", "on", "keep-last", and "keep-group", with a terminating NULL to indicate the end of the list.
- **Use**: This variable is used to provide a list of choices for the 'destroy-unattached' session option, determining the behavior of tmux when sessions are left without attached clients.


---
### options_table_detach_on_destroy_list
- **Type**: `const char *[]`
- **Description**: The `options_table_detach_on_destroy_list` is a static constant array of strings that represents different options for the behavior of detaching sessions when they are destroyed in the tmux application. The array includes options such as "off", "on", "no-detached", "previous", and "next", with a terminating NULL pointer to indicate the end of the list.
- **Use**: This variable is used to define the possible choices for the 'detach-on-destroy' option in tmux, allowing users to configure how sessions should be handled upon destruction.


---
### options_table_extended_keys_list
- **Type**: ``const char *` array`
- **Description**: The `options_table_extended_keys_list` is a static constant array of strings that represents the possible values for the 'extended-keys' option in the tmux configuration. The array contains three string elements: "off", "on", and "always", followed by a NULL terminator to indicate the end of the list.
- **Use**: This array is used to define the valid choices for the 'extended-keys' option, which determines whether extended key sequences are requested from terminals that support them.


---
### options_table_extended_keys_format_list
- **Type**: `const char *[]`
- **Description**: The `options_table_extended_keys_format_list` is a static constant array of strings that contains the possible formats for extended key sequences. The array includes the formats "csi-u" and "xterm", and is terminated with a NULL pointer to indicate the end of the list.
- **Use**: This variable is used to define the available formats for extended key sequences in the options table, allowing the user to select the desired format for key sequence emission.


---
### options_table_allow_passthrough_list
- **Type**: `const char *[]`
- **Description**: The `options_table_allow_passthrough_list` is a static constant array of strings that defines the possible values for the 'allow-passthrough' option in the tmux configuration. This array includes the options 'off', 'on', and 'all', with a terminating NULL to indicate the end of the list.
- **Use**: This variable is used to provide a predefined set of choices for the 'allow-passthrough' option, which controls whether applications can use escape sequences to bypass tmux.


---
### options_table_status_format_default
- **Type**: `const char *[]`
- **Description**: The `options_table_status_format_default` is a static constant array of strings that defines the default format for the status line in the tmux application. It includes two predefined format strings, `OPTIONS_TABLE_STATUS_FORMAT1` and `OPTIONS_TABLE_STATUS_FORMAT2`, which are used to configure the appearance and content of the status line. The array is terminated with a `NULL` pointer to indicate the end of the list.
- **Use**: This variable is used to provide default status line formats that can be applied to the tmux status line configuration.


---
### options_other_names
- **Type**: `const struct options_name_map[]`
- **Description**: The `options_other_names` variable is an array of structures, each containing a pair of strings. These strings represent alternative names for certain options, specifically converting American English spellings to British English spellings for color-related options in the tmux configuration.
- **Use**: This variable is used to map user-visible option names to their alternative spellings, ensuring compatibility with different English dialects.


---
### options_table
- **Type**: ``const struct options_table_entry[]``
- **Description**: The `options_table` is a global array of `options_table_entry` structures that define various configuration options for a server, session, and window in the tmux application. Each entry in the array specifies an option's name, type, scope, default value, and a description of its purpose. This table serves as the master copy of options with their real types, range limits, and default values, which are used to initialize runtime global options.
- **Use**: This variable is used to define and manage configuration options for tmux, allowing users to customize server, session, and window behaviors.


