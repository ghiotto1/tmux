# Purpose
The provided C source code is part of the `tmux` terminal multiplexer, specifically implementing the functionality to display panes on a client. This code defines the command `display-panes`, which is used to visually represent the different panes within a tmux session, allowing users to easily identify and select them. The command is registered with a set of options and flags, and it includes functions for parsing arguments, executing the command, and handling user interactions such as key presses.

The core functionality revolves around rendering pane identifiers on the terminal screen, which involves calculating the visibility and position of each pane, and drawing the pane numbers or letters in a visually distinct manner. The code also manages user input to select a pane by its displayed identifier, and it handles the overlay lifecycle, including drawing and freeing resources. The command supports options for customizing the display duration and behavior, and it integrates with the tmux command queue to execute further commands based on user input. This file is a focused implementation of a specific feature within tmux, providing a user interface component that enhances the usability of the terminal multiplexer by making pane management more intuitive.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_display_panes_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_display_panes_entry` is a constant structure of type `cmd_entry` that defines the command 'display-panes' and its alias 'displayp'. It specifies the arguments, usage, flags, and execution function associated with the command.
- **Use**: This variable is used to register and define the behavior of the 'display-panes' command within the application, allowing it to be executed with specific arguments and options.


# Data Structures

---
### cmd_display_panes_data
- **Type**: `struct`
- **Members**:
    - `item`: A pointer to a cmdq_item structure, representing a command queue item.
    - `state`: A pointer to an args_command_state structure, representing the state of command arguments.
- **Description**: The `cmd_display_panes_data` structure is used to store data related to the display of panes in a client session within the tmux application. It contains pointers to a command queue item and a command state, which are used to manage and execute commands related to displaying panes. This structure is integral to the functionality that allows users to view and interact with multiple panes in a tmux session.


# Functions

---
### cmd_display_panes_args_parse<!-- {{#callable:cmd_display_panes_args_parse}} -->
The function `cmd_display_panes_args_parse` returns a specific argument parse type for the display-panes command.
- **Inputs**:
    - `args`: A pointer to a struct of type `args`, which is not used in this function.
    - `idx`: An unsigned integer representing an index, which is not used in this function.
    - `cause`: A pointer to a character pointer, which is not used in this function.
- **Control Flow**:
    - The function is defined as static, meaning it is limited to the file scope.
    - The function takes three parameters, all marked as unused, indicating they are not utilized within the function body.
    - The function directly returns the constant `ARGS_PARSE_COMMANDS_OR_STRING`.
- **Output**:
    - The function returns an enum value of type `args_parse_type`, specifically `ARGS_PARSE_COMMANDS_OR_STRING`.


---
### cmd_display_panes_draw_pane<!-- {{#callable:cmd_display_panes_draw_pane}} -->
The function `cmd_display_panes_draw_pane` is responsible for drawing a specific window pane on the terminal screen, adjusting its visibility and appearance based on the current screen context and pane properties.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure that contains context information for redrawing the screen, including the client, offsets, and dimensions.
    - `wp`: A pointer to a `window_pane` structure representing the pane to be drawn, including its position, size, and associated window.
- **Control Flow**:
    - Check if the pane is outside the visible area of the screen context and return if it is not visible.
    - Calculate the visible portion of the pane by adjusting offsets and sizes based on the screen context's offsets and dimensions.
    - Adjust the vertical offset if the status line is at the top of the screen.
    - Determine the pane index and format it into a string buffer.
    - Retrieve color settings for the pane from session options and set the foreground and background colors based on whether the pane is active.
    - Format the pane's size into a string buffer and determine if a letter representation is needed for the pane index.
    - If the pane is too small to display detailed information, draw a simplified representation with the pane index and optional letter.
    - For larger panes, draw a detailed representation using a grid pattern to represent the pane index as a clock-like display.
    - If there is enough vertical space, draw additional information such as the pane's size and letter representation.
    - Reset the cursor position to the top-left corner of the terminal.
- **Output**:
    - The function does not return a value; it directly manipulates the terminal display to render the pane's representation.
- **Functions called**:
    - [`window_pane_index`](window.c.driver.md#window_pane_index)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`tty_attributes`](tty.c.driver.md#tty_attributes)
    - [`tty_cursor`](tty.c.driver.md#tty_cursor)
    - [`tty_putn`](tty.c.driver.md#tty_putn)
    - [`tty_putc`](tty.c.driver.md#tty_putc)


---
### cmd_display_panes_draw<!-- {{#callable:cmd_display_panes_draw}} -->
The `cmd_display_panes_draw` function iterates over all panes in the current window of a client session and draws each visible pane using a specified drawing context.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the panes are being drawn.
    - `data`: An unused parameter, typically used for additional data, but not utilized in this function.
    - `ctx`: A pointer to a `screen_redraw_ctx` structure that provides the context for redrawing the screen, including information about the client and its display.
- **Control Flow**:
    - Retrieve the current window from the client's session.
    - Log a debug message with the function name, client name, and window ID.
    - Iterate over each pane in the current window using a TAILQ_FOREACH loop.
    - For each pane, check if it is visible using the [`window_pane_visible`](window.c.driver.md#window_pane_visible) function.
    - If the pane is visible, call [`cmd_display_panes_draw_pane`](#cmd_display_panes_draw_pane) with the drawing context and the pane.
- **Output**:
    - The function does not return a value; it performs actions to draw visible panes on the client's display.
- **Functions called**:
    - [`window_pane_visible`](window.c.driver.md#window_pane_visible)
    - [`cmd_display_panes_draw_pane`](#cmd_display_panes_draw_pane)


---
### cmd_display_panes_free<!-- {{#callable:cmd_display_panes_free}} -->
The `cmd_display_panes_free` function releases resources associated with the display panes command by continuing any pending command queue item and freeing allocated memory.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `data`: A pointer to a `cmd_display_panes_data` structure containing the data to be freed.
- **Control Flow**:
    - Cast the `data` pointer to a `cmd_display_panes_data` structure pointer named `cdata`.
    - Check if `cdata->item` is not NULL; if so, call [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue) to continue the command queue item.
    - Call [`args_make_commands_free`](arguments.c.driver.md#args_make_commands_free) to free the command state stored in `cdata->state`.
    - Free the memory allocated for `cdata`.
- **Output**:
    - This function does not return any value; it performs cleanup operations on the provided data.
- **Functions called**:
    - [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue)
    - [`args_make_commands_free`](arguments.c.driver.md#args_make_commands_free)


---
### cmd_display_panes_key<!-- {{#callable:cmd_display_panes_key}} -->
The `cmd_display_panes_key` function processes a key event to select a window pane by its index and execute associated commands in a tmux client.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the tmux client where the key event occurred.
    - `data`: A pointer to `cmd_display_panes_data` structure containing command queue item and state information.
    - `event`: A pointer to `key_event` structure representing the key event that triggered this function.
- **Control Flow**:
    - Check if the key in the event is a digit ('0'-'9') or a letter ('a'-'z') without modifiers to determine the pane index.
    - If the key is a digit, calculate the index by subtracting '0' from the key.
    - If the key is a letter, calculate the index by subtracting 'a' from the key and adding 10.
    - If the key is not a valid digit or letter, return -1 indicating an error.
    - Retrieve the window pane at the calculated index using [`window_pane_at_index`](window.c.driver.md#window_pane_at_index).
    - If the pane is not found, return 1 indicating no action is taken.
    - Unzoom the current window using [`window_unzoom`](window.c.driver.md#window_unzoom).
    - Format the pane ID into a string using [`xasprintf`](xmalloc.c.driver.md#xasprintf).
    - Create a command list using [`args_make_commands`](arguments.c.driver.md#args_make_commands) with the formatted pane ID.
    - If command list creation fails, append an error to the command queue and free the error string.
    - If the command queue item is NULL, create a new command queue item and append it to the client.
    - If the command queue item is not NULL, insert the new command queue item after the existing item.
    - Free the formatted pane ID string.
    - Return 1 indicating successful processing.
- **Output**:
    - The function returns an integer, where 1 indicates successful processing of the key event and -1 indicates an error in processing the key event.
- **Functions called**:
    - [`window_pane_at_index`](window.c.driver.md#window_pane_at_index)
    - [`window_unzoom`](window.c.driver.md#window_unzoom)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`args_make_commands`](arguments.c.driver.md#args_make_commands)
    - [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append)
    - [`cmdq_get_error`](cmd-queue.c.driver.md#cmdq_get_error)
    - [`cmdq_get_command`](cmd-queue.c.driver.md#cmdq_get_command)
    - [`cmdq_get_state`](cmd-queue.c.driver.md#cmdq_get_state)
    - [`cmdq_insert_after`](cmd-queue.c.driver.md#cmdq_insert_after)


---
### cmd_display_panes_exec<!-- {{#callable:cmd_display_panes_exec}} -->
The `cmd_display_panes_exec` function sets up and executes the display of pane overlays on a client in a tmux session, with optional delay and interaction settings.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item being processed.
- **Control Flow**:
    - Retrieve the command arguments using [`cmd_get_args`](cmd.c.driver.md#cmd_get_args) and the target client using [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client).
    - Check if the target client already has an overlay; if so, return `CMD_RETURN_NORMAL`.
    - Determine if a delay is specified with the '-d' argument; if not, use the default from session options.
    - Allocate memory for `cmd_display_panes_data` and prepare command state with [`args_make_commands_prepare`](arguments.c.driver.md#args_make_commands_prepare).
    - Check for the '-N' argument to decide whether to set up the overlay with or without key handling.
    - Call [`server_client_set_overlay`](server-client.c.driver.md#server_client_set_overlay) to set up the overlay with the specified delay and drawing functions.
    - Return `CMD_RETURN_NORMAL` if not waiting, otherwise return `CMD_RETURN_WAIT`.
- **Output**:
    - The function returns an `enum cmd_retval`, which can be `CMD_RETURN_NORMAL` or `CMD_RETURN_WAIT`, indicating the result of the command execution.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`args_strtonum`](arguments.c.driver.md#args_strtonum)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`args_make_commands_prepare`](arguments.c.driver.md#args_make_commands_prepare)
    - [`server_client_set_overlay`](server-client.c.driver.md#server_client_set_overlay)


