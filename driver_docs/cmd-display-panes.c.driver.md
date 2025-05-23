# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to switch between several programs in one terminal. The file defines the functionality for the "display-panes" command, which is used to visually display the layout of panes within a tmux session on a client. The command is implemented as part of the tmux command interface, with the main components being the command entry definition, argument parsing, and execution logic. The command entry is defined with a name, alias, argument specifications, usage instructions, and flags, which are used to integrate the command into the tmux command system.

The core functionality of the file involves rendering the pane layout on the client's terminal, highlighting active panes, and handling user input to select panes. The [`cmd_display_panes_draw_pane`](#cmd_display_panes_draw_pane) function is responsible for drawing individual panes, while [`cmd_display_panes_exec`](#cmd_display_panes_exec) manages the execution of the command, including setting up overlays and handling delays. The file also includes functions for parsing command arguments, handling key events, and cleaning up resources. This code is designed to be integrated into the tmux application, providing a specific feature that enhances user interaction by allowing them to see and select panes visually.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_display_panes_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_display_panes_entry` is a constant structure of type `cmd_entry` that defines the command 'display-panes' and its associated properties. It includes the command's name, alias, argument specifications, usage instructions, flags, and the function to execute the command.
- **Use**: This variable is used to register and define the behavior of the 'display-panes' command within the application, allowing it to be executed with specified arguments and options.


# Data Structures

---
### cmd_display_panes_data
- **Type**: `struct`
- **Members**:
    - `item`: A pointer to a cmdq_item structure, representing a command queue item.
    - `state`: A pointer to an args_command_state structure, representing the state of command arguments.
- **Description**: The `cmd_display_panes_data` structure is used to store data related to the display panes command in the tmux application. It contains pointers to a command queue item and a command state, which are used to manage the execution and state of the display panes command. This structure is integral to handling the display of panes on a client, allowing for the execution of commands and management of their states within the tmux environment.


# Functions

---
### cmd_display_panes_args_parse<!-- {{#callable:cmd_display_panes_args_parse}} -->
The function `cmd_display_panes_args_parse` returns a specific argument parse type for the `display-panes` command in the tmux application.
- **Inputs**:
    - `args`: A pointer to a struct of type `args`, which is not used in this function.
    - `idx`: An unsigned integer representing an index, which is not used in this function.
    - `cause`: A pointer to a character pointer, which is not used in this function.
- **Control Flow**:
    - The function is defined as static, meaning it is limited to the file scope.
    - The function takes three parameters, all marked as unused, indicating they are not utilized within the function body.
    - The function immediately returns a constant value `ARGS_PARSE_COMMANDS_OR_STRING`.
- **Output**:
    - The function returns an enum value of type `args_parse_type`, specifically `ARGS_PARSE_COMMANDS_OR_STRING`.


---
### cmd_display_panes_draw_pane<!-- {{#callable:cmd_display_panes_draw_pane}} -->
The `cmd_display_panes_draw_pane` function is responsible for drawing a specific window pane on the terminal, adjusting its visibility and appearance based on the current screen context and pane properties.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure that contains the context for redrawing the screen, including the client and offset information.
    - `wp`: A pointer to a `window_pane` structure representing the pane to be drawn.
- **Control Flow**:
    - Check if the pane is outside the visible area of the screen context and return if it is not visible.
    - Calculate the visible portion of the pane by adjusting offsets and sizes based on the screen context's offsets and sizes.
    - Adjust the vertical offset if the status line is at the top of the screen.
    - Calculate the center position of the visible pane area for drawing the pane index.
    - Retrieve the pane index and format it into a string buffer.
    - Determine the color settings for the pane based on whether it is the active pane or not.
    - Format the pane's size into a string buffer and calculate additional label information if the pane index is between 10 and 35.
    - If the pane is too small to display the full index, draw a simplified version of the index and label, then exit.
    - Draw the pane index using a grid pattern if the pane is large enough, iterating over each character in the index string and drawing it using a predefined table.
    - If there is enough vertical space, draw the pane's size and additional label information at the appropriate positions.
    - Reset the cursor position to the top-left corner of the terminal.
- **Output**:
    - The function does not return a value; it directly manipulates the terminal display to draw the pane.


---
### cmd_display_panes_draw<!-- {{#callable:cmd_display_panes_draw}} -->
The `cmd_display_panes_draw` function iterates over all panes in the current window of a client and draws each visible pane using a specified drawing context.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the panes are being drawn.
    - `data`: An unused parameter, typically used for additional data, but not utilized in this function.
    - `ctx`: A pointer to a `screen_redraw_ctx` structure that provides the context for redrawing the screen, including information about the client and its display.
- **Control Flow**:
    - Retrieve the current window from the client's session.
    - Log a debug message with the function name, client name, and window ID.
    - Iterate over each pane in the current window using a TAILQ_FOREACH loop.
    - For each pane, check if it is visible using the `window_pane_visible` function.
    - If the pane is visible, call [`cmd_display_panes_draw_pane`](#cmd_display_panes_draw_pane) to draw the pane using the provided context.
- **Output**:
    - The function does not return a value; it performs its operations directly on the provided context and client structures.
- **Functions called**:
    - [`cmd_display_panes_draw_pane`](#cmd_display_panes_draw_pane)


---
### cmd_display_panes_free<!-- {{#callable:cmd_display_panes_free}} -->
The `cmd_display_panes_free` function is responsible for cleaning up resources associated with the display panes command in the tmux client.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `data`: A pointer to a `cmd_display_panes_data` structure that contains the resources to be freed.
- **Control Flow**:
    - Cast the `data` pointer to a `cmd_display_panes_data` structure pointer named `cdata`.
    - Check if `cdata->item` is not NULL; if so, call `cmdq_continue` with `cdata->item` to continue the command queue item.
    - Call `args_make_commands_free` with `cdata->state` to free any command state resources.
    - Free the memory allocated for `cdata` using `free(cdata)`.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided data.


---
### cmd_display_panes_key<!-- {{#callable:cmd_display_panes_key}} -->
The `cmd_display_panes_key` function processes a key event to select a window pane based on a numeric or alphabetic key input and executes a command associated with the selected pane.
- **Inputs**:
    - `c`: A pointer to the client structure representing the current client.
    - `data`: A pointer to the `cmd_display_panes_data` structure containing command queue item and state information.
    - `event`: A pointer to the `key_event` structure representing the key event that triggered this function.
- **Control Flow**:
    - Check if the key in the event is a digit ('0'-'9') and calculate the index accordingly.
    - If the key is not a digit, check if it is a lowercase letter ('a'-'z') without modifiers and calculate the index as 10 plus the letter's position in the alphabet.
    - If the key is neither a digit nor a valid letter, return -1 indicating an error.
    - Retrieve the window pane at the calculated index from the current window of the client's session.
    - If no window pane is found at the index, return 1 indicating no action is taken.
    - Unzoom the window if it is zoomed.
    - Format the window pane ID into a string and store it in `expanded`.
    - Create a command list using the expanded string and the command state from `cdata`.
    - If command list creation fails, append an error message to the command queue and free the error string.
    - If the command queue item is NULL, create a new command queue item and append it to the client's command queue.
    - If the command queue item is not NULL, create a new command queue item with the state of the existing item and insert it after the existing item in the queue.
    - Free the memory allocated for the `expanded` string.
    - Return 1 indicating successful processing.
- **Output**:
    - The function returns an integer value: -1 for an error, 1 for successful processing, or 1 if no action is taken due to an invalid index.


---
### cmd_display_panes_exec<!-- {{#callable:cmd_display_panes_exec}} -->
The `cmd_display_panes_exec` function sets up an overlay on a client to display pane information with optional delay and interaction settings.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args` and the target client using `cmdq_get_target_client`.
    - Check if the target client already has an overlay; if so, return `CMD_RETURN_NORMAL`.
    - Determine the delay for the overlay either from the arguments or from session options.
    - Allocate memory for `cmd_display_panes_data` and prepare command state with `args_make_commands_prepare`.
    - Set up the overlay on the client using `server_client_set_overlay`, choosing between interactive and non-interactive modes based on the presence of the 'N' argument.
    - Return `CMD_RETURN_NORMAL` if not waiting, otherwise return `CMD_RETURN_WAIT`.
- **Output**:
    - The function returns an `enum cmd_retval`, which can be `CMD_RETURN_NORMAL` or `CMD_RETURN_WAIT`, indicating the result of the command execution.


# Function Declarations (Public API)

---
### cmd_display_panes_args_parse<!-- {{#callable_declaration:cmd_display_panes_args_parse}} -->
Parse arguments for the display-panes command.
- **Description**: This function is used to parse the arguments for the 'display-panes' command in the tmux application. It determines the type of arguments that are expected, which can be either commands or strings. This function is typically invoked as part of the command setup process and does not perform any validation or modification of the arguments themselves. It is expected to be used internally within the command registration process and not directly by end users.
- **Inputs**:
    - `args`: A pointer to a struct args, representing the command arguments. This parameter is not used in the function and can be null.
    - `idx`: An unsigned integer representing the index of the argument to parse. This parameter is not used in the function.
    - `cause`: A pointer to a char pointer, intended to store an error message if parsing fails. This parameter is not used in the function and can be null.
- **Output**: Returns ARGS_PARSE_COMMANDS_OR_STRING, indicating the expected argument type.
- **See also**: [`cmd_display_panes_args_parse`](#cmd_display_panes_args_parse)  (Implementation)


---
### cmd_display_panes_exec<!-- {{#callable_declaration:cmd_display_panes_exec}} -->
Displays panes on a client with optional delay and interaction.
- **Description**: This function is used to visually display the panes of a session on a client, optionally allowing user interaction and specifying a delay for the display. It should be called when you want to provide a visual representation of the panes to the user, possibly for selection or informational purposes. The function can be configured to either wait for user interaction or proceed automatically after a specified delay. It is important to ensure that the target client is valid and that the function is not called if an overlay is already active on the client.
- **Inputs**:
    - `self`: A pointer to the command structure. Must not be null and should be properly initialized with the command arguments.
    - `item`: A pointer to the command queue item. Must not be null and is used to retrieve the target client and manage command execution flow.
- **Output**: Returns an enum value indicating the result of the command execution, which can be CMD_RETURN_NORMAL or CMD_RETURN_WAIT, depending on whether the function waits for user interaction.
- **See also**: [`cmd_display_panes_exec`](#cmd_display_panes_exec)  (Implementation)


