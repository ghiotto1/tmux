# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file defines a comprehensive set of configuration options for tmux, including server, session, and window options. These options are organized into tables that specify their types, default values, and constraints, such as range limits and valid choices. The file includes various option types, such as numbers, strings, colors, and choices, and it provides detailed descriptions for each option, explaining their purpose and usage within tmux.

The code also defines several lists of valid choices for specific options, such as cursor styles, status line formats, and pane border styles. Additionally, it includes macros to facilitate the creation of hook options, which are used to define custom actions triggered by specific events in tmux. The file serves as a central repository for tmux's configuration options, ensuring that all options are consistently defined and documented. This structure allows for easy extension and modification of tmux's behavior by users and developers, making it a crucial component of the tmux configuration system.
# Imports and Dependencies

---
- `sys/types.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### options_table_mode_keys_list
- **Type**: `const char *[]`
- **Description**: The `options_table_mode_keys_list` is a static constant array of strings that contains the available key mode options for a software application, specifically 'emacs' and 'vi'. This array is terminated with a NULL pointer to indicate the end of the list.
- **Use**: This variable is used to define the set of key mode options available for user selection in the application.


---
### options_table_clock_mode_style_list
- **Type**: `const char *[]`
- **Description**: The `options_table_clock_mode_style_list` is a static constant array of string literals representing the available clock mode styles. It contains two options: "12" and "24", which correspond to 12-hour and 24-hour clock formats, respectively.
- **Use**: This variable is used to define the available choices for the clock mode style option in the application, allowing users to select between 12-hour and 24-hour time formats.


---
### options_table_status_list
- **Type**: ``const char *` array`
- **Description**: The `options_table_status_list` is a static constant array of strings that represents different status options available in the software. It includes options such as "off", "on", and numeric values from "2" to "5". The array is terminated with a `NULL` pointer to indicate the end of the list.
- **Use**: This variable is used to define the possible values for a status-related option in the software's configuration.


---
### options_table_message_line_list
- **Type**: `const char *[]`
- **Description**: The `options_table_message_line_list` is a static constant array of string literals representing different message line options. It contains the strings "0", "1", "2", "3", "4", and a terminating NULL pointer.
- **Use**: This array is used to define the available choices for the message line option in the configuration of the software.


---
### options_table_status_keys_list
- **Type**: ``const char *` array`
- **Description**: The `options_table_status_keys_list` is a static constant array of strings that contains the key set options for the status line in the tmux configuration. It includes two options: "emacs" and "vi", which represent different key binding styles. The array is terminated with a NULL pointer to indicate the end of the list.
- **Use**: This variable is used to define the available key set options for the status line in tmux, allowing users to choose between emacs and vi key bindings.


---
### options_table_status_justify_list
- **Type**: `const char *[]`
- **Description**: The `options_table_status_justify_list` is a static constant array of strings that defines the possible justification options for the status line in the tmux application. The array includes options such as "left", "centre", "right", and "absolute-centre", with a terminating NULL to indicate the end of the list.
- **Use**: This variable is used to provide a list of valid justification options for configuring the alignment of the status line in tmux.


---
### options_table_status_position_list
- **Type**: ``const char *` array`
- **Description**: The `options_table_status_position_list` is a static constant array of strings that defines the possible positions for the status line in the tmux application. It contains two options: "top" and "bottom", followed by a NULL terminator to indicate the end of the list.
- **Use**: This variable is used to provide a list of valid choices for configuring the position of the status line in tmux.


---
### options_table_bell_action_list
- **Type**: ``const char *` array`
- **Description**: The `options_table_bell_action_list` is a static constant array of strings that defines the possible actions that can be taken in response to a bell alert in a session. The available actions are "none", "any", "current", and "other", with a terminating `NULL` to indicate the end of the list.
- **Use**: This array is used to provide a list of valid choices for the `bell-action` option in session configurations.


---
### options_table_visual_bell_list
- **Type**: ``const char *` array`
- **Description**: The `options_table_visual_bell_list` is a static constant array of strings that defines the possible values for the visual bell option in the tmux configuration. It includes the options "off", "on", and "both", with a terminating NULL to indicate the end of the list.
- **Use**: This array is used to provide a list of valid choices for configuring how visual bell alerts are displayed in tmux sessions.


---
### options_table_cursor_style_list
- **Type**: ``const char *` array`
- **Description**: The `options_table_cursor_style_list` is a static constant array of strings that defines the available cursor styles in the application. It includes styles such as "default", "blinking-block", "block", "blinking-underline", "underline", "blinking-bar", and "bar".
- **Use**: This array is used to provide a list of valid cursor style options for configuration settings within the application.


---
### options_table_pane_scrollbars_list
- **Type**: `const char *[]`
- **Description**: The `options_table_pane_scrollbars_list` is a static constant array of strings that defines the possible states for pane scrollbars in the application. The array contains three options: "off", "modal", and "on", followed by a NULL terminator to indicate the end of the list.
- **Use**: This variable is used to provide a list of valid choices for configuring the state of pane scrollbars within the application.


---
### options_table_pane_scrollbars_position_list
- **Type**: `const char *[]`
- **Description**: The `options_table_pane_scrollbars_position_list` is a static constant array of strings that defines the possible positions for pane scrollbars in the application. It contains two options: "right" and "left", followed by a NULL terminator to indicate the end of the list.
- **Use**: This variable is used to provide a predefined list of choices for configuring the position of pane scrollbars within the application.


---
### options_table_pane_status_list
- **Type**: `const char *[]`
- **Description**: The `options_table_pane_status_list` is a static constant array of strings that defines the possible status positions for a pane in the application. The array contains three string elements: "off", "top", and "bottom", followed by a NULL terminator to indicate the end of the list.
- **Use**: This variable is used to provide a list of valid options for setting the status position of a pane, which can be referenced in the application's configuration or option handling logic.


---
### options_table_pane_border_indicators_list
- **Type**: ``const char *``
- **Description**: The `options_table_pane_border_indicators_list` is a static constant array of strings that defines the possible options for pane border indicators in a terminal multiplexer environment. The options include "off", "colour", "arrows", and "both", with a terminating NULL pointer to indicate the end of the list.
- **Use**: This variable is used to provide a list of choices for configuring how pane borders are visually indicated in the user interface.


---
### options_table_pane_border_lines_list
- **Type**: ``const char *` array`
- **Description**: The `options_table_pane_border_lines_list` is a static constant array of strings that defines the different styles of border lines that can be used for pane borders in the application. The available styles include "single", "double", "heavy", "simple", and "number".
- **Use**: This array is used to provide a list of choices for configuring the appearance of pane borders in the application.


---
### options_table_popup_border_lines_list
- **Type**: `const char *[]`
- **Description**: The `options_table_popup_border_lines_list` is a static constant array of strings that defines the different styles of border lines that can be used for popups. The available styles include "single", "double", "heavy", "simple", "rounded", "padded", and "none". The array is terminated with a NULL pointer to indicate the end of the list.
- **Use**: This variable is used to provide a list of choices for configuring the border style of popups in the application.


---
### options_table_set_clipboard_list
- **Type**: `const char *[]`
- **Description**: The `options_table_set_clipboard_list` is a static constant array of strings that defines the possible values for the clipboard setting in the tmux configuration. The array includes the options 'off', 'external', and 'on', with a terminating NULL pointer to indicate the end of the list.
- **Use**: This variable is used to provide a list of valid choices for the 'set-clipboard' option in tmux, allowing users to configure how tmux interacts with the system clipboard.


---
### options_table_window_size_list
- **Type**: `const char *[]`
- **Description**: The `options_table_window_size_list` is a static constant array of strings that defines the possible values for window size options in the application. It includes options such as "largest", "smallest", "manual", and "latest", with a terminating NULL to indicate the end of the list.
- **Use**: This variable is used to provide a predefined set of choices for configuring window size options within the application.


---
### options_table_remain_on_exit_list
- **Type**: ``const char *` array`
- **Description**: The `options_table_remain_on_exit_list` is a static constant array of strings that represents the possible values for the 'remain-on-exit' option in the tmux configuration. This option determines the behavior of panes when the program inside them exits, with possible values being 'off', 'on', and 'failed', followed by a NULL terminator to indicate the end of the list.
- **Use**: This array is used to define the valid choices for the 'remain-on-exit' option, which controls whether panes should remain open or be automatically closed upon program exit.


---
### options_table_destroy_unattached_list
- **Type**: `const char *[]`
- **Description**: The `options_table_destroy_unattached_list` is a static constant array of strings that defines possible options for handling sessions that have no attached clients in the tmux application. The options include "off", "on", "keep-last", and "keep-group", with a terminating NULL to indicate the end of the list.
- **Use**: This variable is used to provide a set of predefined choices for the 'destroy-unattached' session option, allowing users to configure how tmux handles unattached sessions.


---
### options_table_detach_on_destroy_list
- **Type**: `const char *[]`
- **Description**: The `options_table_detach_on_destroy_list` is a static constant array of strings that represents different options for detaching behavior when a session is destroyed in the tmux application. The array includes options such as "off", "on", "no-detached", "previous", and "next", with a terminating NULL pointer to indicate the end of the list.
- **Use**: This variable is used to define the possible choices for the 'detach-on-destroy' option in tmux, allowing users to specify how tmux should handle detaching when a session is destroyed.


---
### options_table_extended_keys_list
- **Type**: ``const char *[]``
- **Description**: The `options_table_extended_keys_list` is a static constant array of strings that contains options for extended key sequences. The array includes the strings "off", "on", and "always", followed by a NULL terminator to indicate the end of the list.
- **Use**: This variable is used to define the possible values for the 'extended-keys' option in the options table, allowing users to specify whether to request extended key sequences from terminals.


---
### options_table_extended_keys_format_list
- **Type**: ``const char *` array`
- **Description**: The `options_table_extended_keys_format_list` is a static constant array of strings that defines the available formats for extended key sequences in the software. It contains two string elements: "csi-u" and "xterm", followed by a NULL terminator to indicate the end of the list.
- **Use**: This variable is used to specify the format of extended key sequences that can be emitted by the software.


---
### options_table_allow_passthrough_list
- **Type**: `const char *[]`
- **Description**: The `options_table_allow_passthrough_list` is a static constant array of strings that defines the possible values for the 'allow-passthrough' option in the tmux configuration. This array includes the options 'off', 'on', and 'all', with a terminating NULL pointer to indicate the end of the list.
- **Use**: This variable is used to provide a list of valid choices for the 'allow-passthrough' option, which controls whether applications can bypass tmux using escape sequences.


---
### options_table_status_format_default
- **Type**: `const char *[]`
- **Description**: The `options_table_status_format_default` is a static constant array of strings that defines the default status line formats for a session in the tmux application. It contains two predefined format strings, `OPTIONS_TABLE_STATUS_FORMAT1` and `OPTIONS_TABLE_STATUS_FORMAT2`, followed by a `NULL` terminator. These format strings are used to configure the appearance and content of the status line in tmux.
- **Use**: This variable is used to provide default format settings for the status line in tmux sessions, allowing users to see information about their current session and windows.


---
### options_other_names
- **Type**: ``const struct options_name_map[]``
- **Description**: The `options_other_names` variable is an array of structures of type `options_name_map`. Each element in the array contains a pair of strings, where the first string is an option name using American English spelling, and the second string is the equivalent option name using British English spelling. The array is terminated by a pair of `NULL` values.
- **Use**: This variable is used to map option names between American and British English spellings for compatibility purposes.


---
### options_table
- **Type**: `const struct options_table_entry[]`
- **Description**: The `options_table` is an array of `options_table_entry` structures that define various configuration options for a server, session, and window in the tmux application. Each entry in the array specifies the name, type, scope, default value, and description of an option, allowing for detailed customization of tmux behavior. The options cover a wide range of functionalities, including key bindings, terminal settings, and session management.
- **Use**: This variable is used to store and define the default settings and constraints for tmux options, which can be modified by the user to customize their tmux environment.


