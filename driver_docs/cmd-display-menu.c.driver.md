# Purpose
This C source code file is part of the `tmux` project, a terminal multiplexer that allows users to switch between several programs in one terminal. The file defines functionality for displaying menus and popups within a `tmux` client. It includes two primary command entries: `cmd_display_menu_entry` and `cmd_display_popup_entry`, which correspond to the `display-menu` and `display-popup` commands, respectively. These commands are used to create interactive menus and popups that can be displayed to the user, allowing for enhanced user interaction within the `tmux` environment.

The file contains detailed implementations of the command execution functions [`cmd_display_menu_exec`](#cmd_display_menu_exec) and [`cmd_display_popup_exec`](#cmd_display_popup_exec), which handle the logic for rendering these UI components based on various parameters such as position, size, style, and content. The code also includes helper functions like [`cmd_display_menu_args_parse`](#cmd_display_menu_args_parse) and [`cmd_display_menu_get_position`](#cmd_display_menu_get_position) to parse command arguments and calculate the appropriate positions for displaying the menus and popups. This file is integral to the user interface aspect of `tmux`, providing a mechanism for users to interact with the system through graphical elements like menus and popups, enhancing the overall usability of the terminal multiplexer.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_display_menu_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_display_menu_entry` is a constant structure of type `cmd_entry` that defines the properties and behavior of the 'display-menu' command in the application. It includes fields such as the command name, alias, argument specifications, usage instructions, target specifications, flags, and the execution function.
- **Use**: This variable is used to register and define the behavior of the 'display-menu' command, allowing it to be executed with specific arguments and options within the application.


---
### cmd_display_popup_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_display_popup_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'display-popup' command in the tmux application. It includes details such as the command name, alias, argument specifications, usage instructions, target specifications, flags, and the execution function associated with the command.
- **Use**: This variable is used to register and define the behavior of the 'display-popup' command within the tmux application, allowing users to display a popup window with specified parameters.


# Functions

---
### cmd_display_menu_args_parse<!-- {{#callable:cmd_display_menu_args_parse}} -->
The function `cmd_display_menu_args_parse` determines the type of argument parsing required for a given index in a list of arguments.
- **Inputs**:
    - `args`: A pointer to a struct of type `args`, which contains the list of arguments to be parsed.
    - `idx`: An unsigned integer representing the index of the argument to be parsed.
    - `cause`: A pointer to a character array, which is unused in this function.
- **Control Flow**:
    - Initialize a local variable `i` to 0 and `type` to `ARGS_PARSE_STRING`.
    - Enter an infinite loop to iterate over the arguments.
    - Set `type` to `ARGS_PARSE_STRING` at the start of each loop iteration.
    - Check if the current index `i` is equal to `idx`; if so, break the loop.
    - Retrieve the argument string at index `i` using [`args_string`](arguments.c.driver.md#args_string) and check if it is an empty string; if so, increment `i` and continue to the next iteration.
    - Set `type` to `ARGS_PARSE_STRING` and increment `i`; check if `i` is equal to `idx`, and if so, break the loop.
    - Set `type` to `ARGS_PARSE_COMMANDS_OR_STRING` and increment `i`; check if `i` is equal to `idx`, and if so, break the loop.
    - Return the `type` variable, which indicates the type of parsing required for the argument at index `idx`.
- **Output**:
    - The function returns an `enum args_parse_type` value, which indicates the type of parsing required for the argument at the specified index.
- **Functions called**:
    - [`args_string`](arguments.c.driver.md#args_string)


---
### cmd_display_menu_get_position<!-- {{#callable:cmd_display_menu_get_position}} -->
The function `cmd_display_menu_get_position` calculates the position for displaying a popup menu on a terminal client based on various parameters and conditions.
- **Inputs**:
    - `tc`: A pointer to the `client` structure representing the terminal client where the menu will be displayed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item associated with the current command.
    - `args`: A pointer to the `args` structure containing the command-line arguments passed to the function.
    - `px`: A pointer to an unsigned integer where the calculated x-coordinate for the popup menu will be stored.
    - `py`: A pointer to an unsigned integer where the calculated y-coordinate for the popup menu will be stored.
    - `w`: An unsigned integer representing the width of the popup menu.
    - `h`: An unsigned integer representing the height of the popup menu.
- **Control Flow**:
    - Check if the popup menu dimensions exceed the terminal size; if so, return 0 to indicate failure.
    - Create a format tree from the target item and add mouse position if available.
    - Determine the status line position and adjust the popup position accordingly if status lines are present.
    - Calculate the center position of the popup based on the terminal size and add it to the format tree.
    - Adjust the popup position relative to the mouse if a valid mouse event is present.
    - Calculate the popup position within the window pane and add it to the format tree.
    - Expand the horizontal position based on the '-x' argument and update the x-coordinate pointer.
    - Expand the vertical position based on the '-y' argument and update the y-coordinate pointer.
    - Free the format tree and return 1 to indicate success.
- **Output**:
    - The function returns an integer value: 1 if the position is successfully calculated and set, or 0 if the popup dimensions exceed the terminal size.
- **Functions called**:
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`cmdq_get_event`](cmd-queue.c.driver.md#cmdq_get_event)
    - [`format_create_from_target`](format.c.driver.md#format_create_from_target)
    - [`status_at_line`](status.c.driver.md#status_at_line)
    - [`status_line_size`](status.c.driver.md#status_line_size)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`tty_window_offset`](tty.c.driver.md#tty_window_offset)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`format_free`](format.c.driver.md#format_free)


---
### cmd_display_menu_exec<!-- {{#callable:cmd_display_menu_exec}} -->
The `cmd_display_menu_exec` function displays a menu on a client with specified options and handles user interactions.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve command arguments, target state, event, and target client from the command queue item.
    - Check if the target client already has an overlay; if so, return `CMD_RETURN_NORMAL`.
    - Determine the starting choice for the menu if the '-C' argument is provided, handling errors if the choice is invalid.
    - Create a menu title from the '-T' argument or use an empty string if not provided.
    - Iterate over the arguments to populate the menu with items, checking for sufficient arguments and handling errors if necessary.
    - Check if the menu is valid and has items; if not, free resources and return appropriate error or normal status.
    - Determine the menu position using [`cmd_display_menu_get_position`](#cmd_display_menu_get_position) and handle errors if positioning fails.
    - Retrieve and validate the border style from the arguments, handling errors if the style is invalid.
    - Set flags for menu behavior based on arguments and event validity.
    - Display the menu using [`menu_display`](menu.c.driver.md#menu_display) and return `CMD_RETURN_WAIT` if successful, otherwise return `CMD_RETURN_NORMAL`.
- **Output**:
    - The function returns an `enum cmd_retval` indicating the result of the menu display operation, which can be `CMD_RETURN_NORMAL`, `CMD_RETURN_ERROR`, or `CMD_RETURN_WAIT`.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`cmdq_get_event`](cmd-queue.c.driver.md#cmdq_get_event)
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`args_count`](arguments.c.driver.md#args_count)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`args_strtonum`](arguments.c.driver.md#args_strtonum)
    - [`format_single_from_target`](format.c.driver.md#format_single_from_target)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`menu_create`](menu.c.driver.md#menu_create)
    - [`args_string`](arguments.c.driver.md#args_string)
    - [`menu_add_item`](menu.c.driver.md#menu_add_item)
    - [`menu_free`](menu.c.driver.md#menu_free)
    - [`key_string_lookup_string`](key-string.c.driver.md#key_string_lookup_string)
    - [`cmd_display_menu_get_position`](#cmd_display_menu_get_position)
    - [`options_get`](options.c.driver.md#options_get)
    - [`options_find_choice`](options.c.driver.md#options_find_choice)
    - [`options_table_entry`](tmux.h.driver.md#options_table_entry)
    - [`menu_display`](menu.c.driver.md#menu_display)


---
### cmd_display_popup_exec<!-- {{#callable:cmd_display_popup_exec}} -->
The `cmd_display_popup_exec` function displays a popup window on a client with customizable dimensions, styles, and commands.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve command arguments and target session and client information.
    - Check if the 'C' argument is present to clear any existing overlay on the client and return normal.
    - Determine the height and width of the popup, using default values or percentages if specified by arguments.
    - Ensure the popup dimensions do not exceed the terminal size.
    - Calculate the popup's position using [`cmd_display_menu_get_position`](#cmd_display_menu_get_position).
    - Determine the border style and lines for the popup based on arguments or session options.
    - Set the current working directory for the popup, defaulting to the client's current directory if not specified.
    - Determine the shell command to execute in the popup, defaulting to the session's default command or shell if not specified.
    - Create an environment for the popup if 'e' arguments are provided.
    - Set the popup title based on the 'T' argument or default to an empty string.
    - Set flags for the popup based on the 'E' argument to control exit behavior.
    - Call [`popup_display`](popup.c.driver.md#popup_display) to render the popup with the specified parameters.
    - Free allocated resources such as environment, current working directory, title, and argument vector.
    - Return `CMD_RETURN_WAIT` to indicate the command is waiting for the popup to close.
- **Output**:
    - Returns an `enum cmd_retval` indicating the result of the command execution, either `CMD_RETURN_NORMAL` or `CMD_RETURN_WAIT`.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`args_count`](arguments.c.driver.md#args_count)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`server_client_clear_overlay`](server-client.c.driver.md#server_client_clear_overlay)
    - [`args_percentage`](arguments.c.driver.md#args_percentage)
    - [`cmd_display_menu_get_position`](#cmd_display_menu_get_position)
    - [`options_get`](options.c.driver.md#options_get)
    - [`options_find_choice`](options.c.driver.md#options_find_choice)
    - [`options_table_entry`](tmux.h.driver.md#options_table_entry)
    - [`format_single_from_target`](format.c.driver.md#format_single_from_target)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`server_client_get_cwd`](server-client.c.driver.md#server_client_get_cwd)
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`args_string`](arguments.c.driver.md#args_string)
    - [`checkshell`](tmux.c.driver.md#checkshell)
    - [`cmd_append_argv`](cmd.c.driver.md#cmd_append_argv)
    - [`args_to_vector`](arguments.c.driver.md#args_to_vector)
    - [`environ_create`](environ.c.driver.md#environ_create)
    - [`args_first_value`](arguments.c.driver.md#args_first_value)
    - [`environ_put`](environ.c.driver.md#environ_put)
    - [`args_next_value`](arguments.c.driver.md#args_next_value)
    - [`popup_display`](popup.c.driver.md#popup_display)
    - [`cmd_free_argv`](cmd.c.driver.md#cmd_free_argv)
    - [`environ_free`](environ.c.driver.md#environ_free)


