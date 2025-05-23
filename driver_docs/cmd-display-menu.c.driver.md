# Purpose
This C source code file is part of the `tmux` project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file defines functionality for displaying interactive menus and popups within a `tmux` client. It includes two primary command entries: `cmd_display_menu_entry` and `cmd_display_popup_entry`, which correspond to the `display-menu` and `display-popup` commands, respectively. These commands allow users to create and interact with menus and popups, specifying various options such as position, size, style, and associated commands or shell scripts.

The file contains detailed implementations of the command execution functions [`cmd_display_menu_exec`](#cmd_display_menu_exec) and [`cmd_display_popup_exec`](#cmd_display_popup_exec), which handle the logic for rendering the menus and popups on the terminal screen. It also includes helper functions like [`cmd_display_menu_args_parse`](#cmd_display_menu_args_parse) and [`cmd_display_menu_get_position`](#cmd_display_menu_get_position) to parse command arguments and calculate the appropriate positions for displaying the UI elements. The code is structured to integrate with the broader `tmux` system, utilizing existing structures and functions to manage client sessions, handle user input, and format output. This file is a crucial component of the `tmux` user interface, enhancing the interactivity and usability of the terminal multiplexer by providing dynamic, customizable menu and popup features.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_display_menu_entry
- **Type**: `struct cmd_entry`
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
    - Check if `i` equals `idx`; if true, break the loop.
    - Retrieve the argument string at index `i` using `args_string` and increment `i`.
    - If the retrieved argument string is empty, continue to the next iteration of the loop.
    - Set `type` to `ARGS_PARSE_STRING` and increment `i`; if `i` equals `idx`, break the loop.
    - Set `type` to `ARGS_PARSE_COMMANDS_OR_STRING` and increment `i`; if `i` equals `idx`, break the loop.
    - Return the `type` variable.
- **Output**:
    - The function returns an `enum args_parse_type` which indicates the type of parsing required for the argument at the specified index.


---
### cmd_display_menu_get_position<!-- {{#callable:cmd_display_menu_get_position}} -->
The function `cmd_display_menu_get_position` calculates the position of a popup menu on a terminal client based on various parameters and conditions.
- **Inputs**:
    - `tc`: A pointer to the client structure representing the terminal client where the menu will be displayed.
    - `item`: A pointer to the command queue item structure, which contains context information for the command execution.
    - `args`: A pointer to the arguments structure, which holds the command-line arguments passed to the function.
    - `px`: A pointer to an unsigned integer where the calculated x-coordinate of the popup menu will be stored.
    - `py`: A pointer to an unsigned integer where the calculated y-coordinate of the popup menu will be stored.
    - `w`: An unsigned integer representing the width of the popup menu.
    - `h`: An unsigned integer representing the height of the popup menu.
- **Control Flow**:
    - Check if the popup dimensions exceed the terminal size; if so, return 0 to indicate failure.
    - Create a format tree from the target item and add mouse position if available.
    - Determine the status line position and adjust the popup position accordingly if status lines are present.
    - Calculate the center position of the popup within the terminal and add it to the format tree.
    - Adjust the popup position relative to the mouse if a valid mouse event is present.
    - Calculate the popup position within the window pane and add it to the format tree.
    - Expand the horizontal position based on the '-x' argument and update the x-coordinate pointer.
    - Expand the vertical position based on the '-y' argument and update the y-coordinate pointer.
    - Free the format tree and return 1 to indicate success.
- **Output**:
    - The function returns an integer value: 1 if the position is successfully calculated and set, or 0 if the popup dimensions exceed the terminal size.


---
### cmd_display_menu_exec<!-- {{#callable:cmd_display_menu_exec}} -->
The `cmd_display_menu_exec` function displays a menu on a client in the tmux environment based on the provided arguments and configurations.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve command arguments, target state, event, and target client from the provided structures.
    - Check if the target client already has an overlay; if so, return `CMD_RETURN_NORMAL`.
    - Determine the starting choice for the menu if the 'C' argument is provided, handling errors if the value is invalid.
    - Create a menu title from the 'T' argument if provided, or use an empty string otherwise.
    - Create a menu and populate it with items based on the arguments, handling errors if arguments are insufficient or invalid.
    - Check if the menu is valid and has items; if not, free resources and return appropriate error or normal status.
    - Determine the menu position using [`cmd_display_menu_get_position`](#cmd_display_menu_get_position), and handle errors if positioning fails.
    - Retrieve and validate the border style from the arguments, handling errors if the style is invalid.
    - Set flags for menu behavior based on the presence of 'O' and 'M' arguments.
    - Display the menu using `menu_display` with the calculated parameters and return `CMD_RETURN_WAIT` if successful, or `CMD_RETURN_NORMAL` otherwise.
- **Output**:
    - The function returns an `enum cmd_retval` indicating the result of the menu display operation, which can be `CMD_RETURN_NORMAL`, `CMD_RETURN_ERROR`, or `CMD_RETURN_WAIT`.
- **Functions called**:
    - [`cmd_display_menu_get_position`](#cmd_display_menu_get_position)
    - [`options_table_entry`](tmux.h.driver.md#options_table_entry)


---
### cmd_display_popup_exec<!-- {{#callable:cmd_display_popup_exec}} -->
The `cmd_display_popup_exec` function displays a popup window on a client with customizable dimensions, styles, and commands.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve command arguments and target session and client information.
    - Check if the 'C' argument is present to clear any existing overlay on the client and return normal.
    - Determine the height and width of the popup, using default values or percentages from arguments, and handle any errors in percentage parsing.
    - Ensure the popup dimensions do not exceed the terminal size and calculate the popup's position.
    - Determine the border style and lines for the popup based on arguments and options, handling any errors in choice parsing.
    - Set the current working directory for the popup, defaulting to the client's current directory if not specified.
    - Determine the shell command to execute in the popup, defaulting to the session's default command or shell if not provided.
    - Create an environment for the popup if 'e' arguments are provided, adding each environment variable to the environment.
    - Set the popup title based on the 'T' argument or default to an empty string.
    - Set flags for the popup based on the 'E' argument, indicating whether the popup should close on exit or exit with zero.
    - Call `popup_display` to render the popup with the specified parameters, and handle cleanup of resources if the display fails.
    - Free allocated resources such as environment, current working directory, title, and argument vector.
    - Return `CMD_RETURN_WAIT` to indicate the command should wait for the popup to close.
- **Output**:
    - Returns an `enum cmd_retval` indicating the result of the command execution, either `CMD_RETURN_NORMAL` or `CMD_RETURN_WAIT`.
- **Functions called**:
    - [`cmd_display_menu_get_position`](#cmd_display_menu_get_position)
    - [`options_table_entry`](tmux.h.driver.md#options_table_entry)


# Function Declarations (Public API)

---
### cmd_display_menu_args_parse<!-- {{#callable_declaration:cmd_display_menu_args_parse}} -->
Determines the type of argument at a specified index.
- **Description**: This function is used to determine the type of argument at a given index within a list of arguments. It is typically called during the parsing of command-line arguments to identify whether an argument is a string or a command. The function iterates through the arguments until it reaches the specified index, returning the type of the argument at that position. It is important to ensure that the index provided is within the bounds of the argument list to avoid undefined behavior.
- **Inputs**:
    - `args`: A pointer to a struct args, representing the list of arguments to be parsed. Must not be null.
    - `idx`: An unsigned integer representing the index of the argument whose type is to be determined. Must be within the range of available arguments.
    - `cause`: A pointer to a char pointer, which is unused in this function. Can be null.
- **Output**: Returns an enum args_parse_type indicating the type of the argument at the specified index, which can be either ARGS_PARSE_STRING or ARGS_PARSE_COMMANDS_OR_STRING.
- **See also**: [`cmd_display_menu_args_parse`](#cmd_display_menu_args_parse)  (Implementation)


---
### cmd_display_menu_exec<!-- {{#callable_declaration:cmd_display_menu_exec}} -->
Displays a menu on a client.
- **Description**: This function is used to display a menu on a specified client within the tmux environment. It should be called when a menu needs to be presented to the user, with options for customization such as styles and starting choice. The function requires a valid command and command queue item, and it handles various menu configurations through arguments. It is important to ensure that the target client does not already have an overlay before calling this function, as it will return immediately in such cases. The function also manages error conditions, such as invalid starting choices or insufficient arguments, by returning appropriate error codes.
- **Inputs**:
    - `self`: A pointer to the command structure. Must not be null and should contain valid command arguments.
    - `item`: A pointer to the command queue item. Must not be null and should represent the current command execution context.
- **Output**: Returns an enum cmd_retval indicating the result of the menu display operation, such as CMD_RETURN_NORMAL or CMD_RETURN_ERROR.
- **See also**: [`cmd_display_menu_exec`](#cmd_display_menu_exec)  (Implementation)


---
### cmd_display_popup_exec<!-- {{#callable_declaration:cmd_display_popup_exec}} -->
Displays a popup window on a client.
- **Description**: This function is used to display a popup window on a specified client within a session. It can be customized with various options such as size, position, style, and command execution. The function should be called when a popup needs to be shown, and it handles the necessary setup and display logic. Preconditions include having a valid client and session context. The function manages resources like environment variables and command arguments, ensuring they are properly allocated and freed.
- **Inputs**:
    - `self`: A pointer to the command structure representing the current command. Must not be null.
    - `item`: A pointer to the command queue item associated with the command execution. Must not be null.
- **Output**: Returns an enum cmd_retval indicating the result of the command execution, such as CMD_RETURN_NORMAL or CMD_RETURN_WAIT.
- **See also**: [`cmd_display_popup_exec`](#cmd_display_popup_exec)  (Implementation)


