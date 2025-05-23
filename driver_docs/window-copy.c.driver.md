# Purpose
The provided C source code is part of the `tmux` terminal multiplexer, specifically handling the "copy mode" functionality. This code is responsible for enabling users to navigate, select, and copy text from the terminal screen buffer. It provides a comprehensive set of functions to manage text selection, scrolling, and cursor movement within a terminal pane. The code defines two main modes: "copy-mode" and "view-mode," each with its initialization, resizing, and command handling logic.

Key components of this code include functions for initializing and freeing resources related to copy mode, handling user commands for text navigation and selection, and managing the visual representation of selected text. The code also supports searching within the terminal buffer, both with plain text and regular expressions, and allows for operations like jumping to specific lines or marks. The code is structured to handle various user interactions, such as mouse events for text selection and keyboard shortcuts for navigation, making it a crucial part of the `tmux` user interface for text manipulation.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `regex.h`
- `stdlib.h`
- `string.h`
- `time.h`
- `tmux.h`


# Global Variables

---
### window_copy_key_table
- **Type**: `function pointer`
- **Description**: `window_copy_key_table` is a static function pointer that returns a constant character pointer. It is used to determine the key table associated with a window mode entry, specifically for copy mode operations in the tmux terminal multiplexer.
- **Use**: This function pointer is used to retrieve the appropriate key table for handling key bindings in copy mode.


---
### window_copy_init
- **Type**: ``static struct screen *``
- **Description**: The `window_copy_init` variable is a static function pointer that returns a pointer to a `struct screen`. It is used to initialize a screen for the copy mode in a window.
- **Use**: This variable is used to set up the initial state of a screen when entering copy mode in a window.


---
### window_copy_view_init
- **Type**: `static struct screen *`
- **Description**: `window_copy_view_init` is a static function that initializes a screen structure for the 'view-mode' in a window pane. It sets up the necessary data structures and context for viewing content in a non-editable mode.
- **Use**: This function is used to set up the screen and related data for a window pane when entering view mode, allowing the user to view content without modifying it.


---
### window_copy_get_screen
- **Type**: `struct screen *`
- **Description**: The `window_copy_get_screen` variable is a static function pointer that returns a pointer to a `struct screen`. It is used within the context of window copy mode in the tmux codebase.
- **Use**: This variable is used to retrieve the screen associated with a given window mode entry, typically to access or manipulate the screen's content during copy operations.


---
### window_copy_match_at_cursor
- **Type**: `function pointer`
- **Description**: `window_copy_match_at_cursor` is a static function pointer that takes a pointer to a `window_copy_mode_data` structure as an argument and returns a character pointer. This function is likely used to find a match at the current cursor position within the copy mode data.
- **Use**: This function is used to retrieve a match at the current cursor position in the window copy mode.


---
### window_copy_stringify
- **Type**: `function pointer`
- **Description**: `window_copy_stringify` is a static function pointer that takes several parameters including a pointer to a `grid` structure, and unsigned integers representing positions and lengths. It returns a pointer to a character string.
- **Use**: This function is used to convert a portion of a grid into a string representation, likely for copying or displaying text from a grid structure.


---
### window_copy_get_selection
- **Type**: `static void *`
- **Description**: The `window_copy_get_selection` function is a static function that returns a pointer to a selection within a window copy mode entry. It takes a pointer to a `window_mode_entry` structure and a pointer to a `size_t` variable as arguments.
- **Use**: This function is used to retrieve the current selection from a window copy mode entry, which can then be used for operations like copying or processing the selected text.


---
### window_copy_mode
- **Type**: ``const struct window_mode``
- **Description**: The `window_copy_mode` is a constant structure of type `window_mode` that defines the behavior and properties of the 'copy-mode' in a window. It includes function pointers for initialization, freeing resources, resizing, handling key tables, executing commands, formatting, and getting the screen associated with the mode.
- **Use**: This variable is used to define and manage the 'copy-mode' functionality within a window, allowing for operations like copying text from a window pane.


---
### window_view_mode
- **Type**: ``const struct window_mode``
- **Description**: The `window_view_mode` is a constant structure of type `window_mode` that defines the behavior and properties of a 'view-mode' in a windowing system. It includes function pointers for initialization, freeing resources, resizing, handling key tables, executing commands, formatting, and getting the screen associated with the mode.
- **Use**: This variable is used to define and manage the 'view-mode' of a window, providing specific functions for its lifecycle and interactions.


---
### window_copy_cmd_table
- **Type**: `array of structs`
- **Description**: The `window_copy_cmd_table` is a static constant array of structures, each containing information about a command in the window copy mode. Each structure includes the command name, minimum and maximum arguments, argument parsing details, a clear action enum, and a function pointer to the command's implementation.
- **Use**: This variable is used to define and manage the set of commands available in the window copy mode, allowing for command execution based on user input.


# Data Structures

---
### window_copy_cmd_action
- **Type**: `enum`
- **Members**:
    - `WINDOW_COPY_CMD_NOTHING`: Represents no action to be taken.
    - `WINDOW_COPY_CMD_REDRAW`: Indicates that the screen should be redrawn.
    - `WINDOW_COPY_CMD_CANCEL`: Signals to cancel the current operation.
- **Description**: The `window_copy_cmd_action` is an enumeration that defines possible actions that can be taken in response to a command in the window copy mode of the tmux application. It includes actions such as doing nothing, redrawing the screen, or canceling the current operation. This enum is used to control the flow of command execution and screen updates in the copy mode.


---
### window_copy_cmd_clear
- **Type**: `enum`
- **Members**:
    - `WINDOW_COPY_CMD_CLEAR_ALWAYS`: Indicates that the command should always clear the selection.
    - `WINDOW_COPY_CMD_CLEAR_NEVER`: Indicates that the command should never clear the selection.
    - `WINDOW_COPY_CMD_CLEAR_EMACS_ONLY`: Indicates that the command should clear the selection only in Emacs mode.
- **Description**: The `window_copy_cmd_clear` enum defines the behavior for clearing selections in a window copy mode. It specifies three possible states: always clear the selection, never clear it, or clear it only when in Emacs mode. This enum is used to control how different commands interact with the selection state in the window copy mode.


---
### window_copy_cmd_state
- **Type**: `struct`
- **Members**:
    - `wme`: A pointer to a `window_mode_entry` structure, representing the mode entry associated with the window copy command state.
    - `args`: A pointer to an `args` structure, representing the arguments passed to the window copy command.
    - `wargs`: A pointer to an `args` structure, representing the working arguments for the window copy command.
    - `m`: A pointer to a `mouse_event` structure, representing the mouse event associated with the window copy command.
    - `c`: A pointer to a `client` structure, representing the client associated with the window copy command state.
    - `s`: A pointer to a `session` structure, representing the session associated with the window copy command state.
    - `wl`: A pointer to a `winlink` structure, representing the window link associated with the window copy command state.
- **Description**: The `window_copy_cmd_state` structure is used to maintain the state of a command in the window copy mode of a terminal multiplexer like tmux. It holds references to various components such as the mode entry, arguments, mouse events, client, session, and window link, which are necessary for executing and managing commands within the window copy mode. This structure facilitates the interaction between different parts of the window copy mode, allowing commands to be processed and executed with the relevant context and data.


---
### window_copy_mode_data
- **Type**: `struct`
- **Members**:
    - `screen`: A screen structure representing the visible screen in copy mode.
    - `backing`: A pointer to the full content of the copy-mode grid, either the original pane or a command output.
    - `backing_written`: An integer indicating if the backing display has started.
    - `writing`: A pointer to a screen structure used for writing output.
    - `ictx`: A pointer to an input context structure.
    - `viewmode`: An integer indicating if view mode is entered.
    - `oy`: An unsigned integer representing the number of lines scrolled up.
    - `selx`: An unsigned integer for the beginning x-coordinate of the selection.
    - `sely`: An unsigned integer for the beginning y-coordinate of the selection.
    - `endselx`: An unsigned integer for the end x-coordinate of the selection.
    - `endsely`: An unsigned integer for the end y-coordinate of the selection.
    - `cursordrag`: An enum indicating the cursor drag state for selection synchronization.
    - `modekeys`: An integer representing the mode keys setting.
    - `lineflag`: An enum indicating the line selection mode.
    - `rectflag`: An integer indicating if rectangle copy mode is active.
    - `scroll_exit`: An integer indicating if the mode should exit on scroll to end.
    - `hide_position`: An integer indicating if the position marker should be hidden.
    - `selflag`: An enum indicating the selection mode (character, word, or line).
    - `separators`: A constant character pointer to the word separators.
    - `dx`: An unsigned integer for the drag start x-coordinate.
    - `dy`: An unsigned integer for the drag start y-coordinate.
    - `selrx`: An unsigned integer for the selection reset x-coordinate.
    - `selry`: An unsigned integer for the selection reset y-coordinate.
    - `endselrx`: An unsigned integer for the end selection reset x-coordinate.
    - `endselry`: An unsigned integer for the end selection reset y-coordinate.
    - `cx`: An unsigned integer for the current x-coordinate of the cursor.
    - `cy`: An unsigned integer for the current y-coordinate of the cursor.
    - `lastcx`: An unsigned integer for the x-coordinate in the last line with content.
    - `lastsx`: An unsigned integer for the size of the last line with content.
    - `mx`: An unsigned integer for the mark x-coordinate.
    - `my`: An unsigned integer for the mark y-coordinate.
    - `showmark`: An integer indicating if the mark should be shown.
    - `searchtype`: An integer representing the type of search being performed.
    - `searchdirection`: An integer indicating the direction of the search.
    - `searchregex`: An integer indicating if the search is using regex.
    - `searchstr`: A character pointer to the search string.
    - `searchmark`: An unsigned character pointer to the search mark data.
    - `searchcount`: An integer representing the count of search results.
    - `searchmore`: An integer indicating if more search results are available.
    - `searchall`: An integer indicating if all search results should be found.
    - `searchx`: An integer for the x-coordinate of the search start position.
    - `searchy`: An integer for the y-coordinate of the search start position.
    - `searcho`: An integer for the offset of the search start position.
    - `searchgen`: An unsigned character for the search generation.
    - `timeout`: An integer indicating if the search has timed out.
    - `jumptype`: An integer representing the type of jump being performed.
    - `jumpchar`: A pointer to a UTF-8 data structure for the jump character.
    - `dragtimer`: An event structure for handling drag timing.
- **Description**: The `window_copy_mode_data` structure is used in the context of a terminal multiplexer to manage the state and behavior of a window in copy mode. It contains fields for managing the screen display, selection coordinates, cursor position, search functionality, and various modes and flags that control how text is selected and displayed. This structure is integral to handling user interactions in copy mode, such as scrolling, selecting text, and searching within the terminal window.


# Functions

---
### window_copy_scroll_timer<!-- {{#callable:window_copy_scroll_timer}} -->
The `window_copy_scroll_timer` function manages the scrolling behavior in a window copy mode by adjusting the cursor position based on the current line and setting a timer for continuous scrolling.
- **Inputs**:
    - `fd`: An unused file descriptor, typically for event handling.
    - `events`: An unused short integer representing event types.
    - `arg`: A pointer to a `window_mode_entry` structure containing the current window mode data.
- **Control Flow**:
    - The function starts by retrieving the `window_mode_entry`, `window_pane`, and `window_copy_mode_data` from the provided argument.
    - It then deletes any existing timer associated with dragging.
    - If the current window mode entry is not the first in the queue of modes for the window pane, the function returns early.
    - If the cursor's vertical position is at the top of the screen, it sets a timer and moves the cursor up by one line.
    - If the cursor's vertical position is at the bottom of the screen, it sets a timer and moves the cursor down by one line.
- **Output**:
    - The function does not return a value; it modifies the state of the window copy mode by potentially moving the cursor and setting a timer for continuous scrolling.
- **Functions called**:
    - [`window_copy_cursor_up`](#window_copy_cursor_up)
    - [`window_copy_cursor_down`](#window_copy_cursor_down)


---
### window_copy_clone_screen<!-- {{#callable:window_copy_clone_screen}} -->
The `window_copy_clone_screen` function creates a new `screen` structure that is a clone of the source screen, optionally trimming empty lines and adjusting cursor positions.
- **Inputs**:
    - `src`: A pointer to the source `screen` structure that is to be cloned.
    - `hint`: A pointer to a hint `screen` structure that provides size information for the new screen.
    - `cx`: A pointer to a `u_int` variable that will hold the x-coordinate of the cursor in the new screen.
    - `cy`: A pointer to a `u_int` variable that will hold the y-coordinate of the cursor in the new screen.
    - `trim`: An integer flag indicating whether to trim empty lines from the bottom of the source screen.
- **Control Flow**:
    - Allocate memory for the new `screen` structure `dst`.
    - Calculate the total height `sy` of the source screen by adding its height and the history size.
    - If `trim` is set, reduce `sy` by checking for empty lines at the bottom of the source screen.
    - Initialize the new screen `dst` with the width of the source screen and the calculated height `sy`.
    - Set the history flag for the grid of the new screen to prevent line deletion during resizing.
    - Duplicate the lines from the source screen's grid to the new screen's grid.
    - Adjust the cursor position in the new screen based on the source screen's cursor position.
    - If `cx` and `cy` pointers are provided, update them with the new cursor coordinates.
    - Check if the width of the hint screen differs from the new screen and handle reflow if necessary.
    - Resize the cursor in the new screen based on the hint dimensions.
    - Return the newly created screen `dst`.
- **Output**:
    - Returns a pointer to the newly created `screen` structure that is a clone of the source screen.


---
### window_copy_common_init<!-- {{#callable:window_copy_common_init}} -->
Initializes the common data structure for window copy mode.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure that contains information about the window mode.
- **Control Flow**:
    - Allocates memory for a new `window_copy_mode_data` structure and assigns it to `wme->data`.
    - Initializes various fields in the `window_copy_mode_data` structure, including cursor drag state, selection flags, and search parameters.
    - If a search string is present in the associated `window_pane`, it sets the search type and duplicates the search string.
    - Initializes the screen for the copy mode with the dimensions of the base screen of the window pane.
    - Sets the default cursor style for the screen.
    - Retrieves the mode keys option from the window's options and assigns it to the modekeys field in the data structure.
    - Sets up a timer for drag events.
- **Output**:
    - Returns a pointer to the initialized `window_copy_mode_data` structure.


---
### window_copy_init<!-- {{#callable:window_copy_init}} -->
Initializes the window copy mode for a given window pane.
- **Inputs**:
    - `wme`: Pointer to the `window_mode_entry` structure representing the current window mode.
    - `fs`: Pointer to the `cmd_find_state` structure, unused in this function.
    - `args`: Pointer to the `args` structure containing command-line arguments.
- **Control Flow**:
    - Calls [`window_copy_common_init`](#window_copy_common_init) to initialize common data for window copy mode.
    - Clones the screen from the base pane to create a backing screen for the copy mode.
    - Calculates the cursor position based on the height of the backing screen.
    - Sets flags for scrolling behavior and visibility of the cursor position.
    - Copies hyperlinks from the base screen if available.
    - Writes the initial content of the screen to the new screen context.
    - Moves the cursor to the calculated position.
- **Output**:
    - Returns a pointer to the initialized `screen` structure representing the copy mode screen.
- **Functions called**:
    - [`window_copy_common_init`](#window_copy_common_init)
    - [`window_copy_clone_screen`](#window_copy_clone_screen)
    - [`window_copy_write_line`](#window_copy_write_line)


---
### window_copy_view_init<!-- {{#callable:window_copy_view_init}} -->
Initializes the view mode for a window copy operation.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure that contains the state of the window mode.
    - `fs`: An unused pointer to a `cmd_find_state` structure.
    - `args`: An unused pointer to an `args` structure.
- **Control Flow**:
    - Retrieve the `window_pane` from the `window_mode_entry` structure.
    - Call [`window_copy_common_init`](#window_copy_common_init) to initialize common data for window copy mode.
    - Set the `viewmode` field of the `window_copy_mode_data` structure to 1.
    - Allocate memory for the `backing` screen and initialize it with the size of the base screen.
    - Allocate memory for the `writing` screen and initialize it with a height of 0.
    - Initialize the input context for handling user input.
    - Set the mouse coordinates based on the current cursor position.
    - Set the `showmark` field to 0.
    - Return a pointer to the initialized screen structure.
- **Output**:
    - Returns a pointer to the initialized `screen` structure associated with the window copy mode.
- **Functions called**:
    - [`window_copy_common_init`](#window_copy_common_init)


---
### window_copy_free<!-- {{#callable:window_copy_free}} -->
The `window_copy_free` function deallocates resources associated with a `window_mode_entry` structure.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` which contains data related to the window copy mode.
- **Control Flow**:
    - The function retrieves the `window_copy_mode_data` structure from the `window_mode_entry`.
    - It cancels any active drag timer associated with the copy mode.
    - It frees various dynamically allocated strings and structures related to search marks, search strings, and jump characters.
    - If the `writing` field is not NULL, it frees the associated screen and memory.
    - If the `ictx` field is not NULL, it frees the input context.
    - It frees the backing screen and the main screen associated with the copy mode.
    - Finally, it frees the `window_copy_mode_data` structure itself.
- **Output**:
    - The function does not return a value; it performs cleanup and deallocation of resources.


---
### window_copy_add<!-- {{#callable:window_copy_add}} -->
The `window_copy_add` function appends formatted text to a window pane in copy mode.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the window pane to which text will be added.
    - `parse`: An integer flag indicating whether the text should be parsed.
    - `fmt`: A format string that specifies how subsequent arguments are converted for output.
    - `ap`: A variable argument list that contains the values to be formatted and added to the window pane.
- **Control Flow**:
    - The function starts by initializing a variable argument list using `va_start` with the format string `fmt`.
    - It then calls the [`window_copy_vadd`](#window_copy_vadd) function, passing the window pane, parse flag, format string, and the argument list.
    - Finally, it ends the variable argument list with `va_end`.
- **Output**:
    - The function does not return a value; it modifies the state of the specified window pane by adding formatted text to it.
- **Functions called**:
    - [`window_copy_vadd`](#window_copy_vadd)


---
### window_copy_init_ctx_cb<!-- {{#callable:window_copy_init_ctx_cb}} -->
Initializes the context for a `tty_ctx` structure by copying default cell values and setting callback pointers to NULL.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure, which is unused in this function.
    - `ttyctx`: A pointer to a `tty_ctx` structure that will be initialized.
- **Control Flow**:
    - The function begins by copying the default cell properties from `grid_default_cell` into the `defaults` field of the `ttyctx` structure.
    - Next, it sets the `palette`, `redraw_cb`, `set_client_cb`, and `arg` fields of the `ttyctx` structure to NULL.
- **Output**:
    - The function does not return a value; it modifies the `ttyctx` structure directly.


---
### window_copy_vadd<!-- {{#callable:window_copy_vadd}} -->
The `window_copy_vadd` function adds formatted text to a window pane in copy mode, handling both parsing and direct output.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to which text will be added.
    - `parse`: An integer flag indicating whether the input should be parsed (1) or not (0).
    - `fmt`: A format string used to specify how the text should be formatted.
    - `ap`: A `va_list` containing the arguments to be formatted according to `fmt`.
- **Control Flow**:
    - If `parse` is true, the function uses `vasprintf` to format the text and then parses it for screen output.
    - The function initializes screen writing contexts for both the backing and writing screens.
    - If the backing has already been written to, it performs a carriage return and line feed before writing new content.
    - Depending on the `parse` flag, it either copies the formatted text from the writing screen to the backing screen or directly writes the formatted text.
    - The function updates the offset for the output history and redraws the necessary lines in the window pane.
- **Output**:
    - The function does not return a value; instead, it modifies the state of the window pane by adding formatted text to it.
- **Functions called**:
    - [`window_copy_redraw_lines`](#window_copy_redraw_lines)


---
### window_copy_scroll<!-- {{#callable:window_copy_scroll}} -->
The `window_copy_scroll` function scrolls the content of a window pane in copy mode.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to be scrolled.
    - `sl_mpos`: An integer representing the position of the slider in the scroll bar.
    - `my`: An unsigned integer representing the mouse Y position.
    - `scroll_exit`: An integer indicating whether to exit scroll mode after scrolling.
- **Control Flow**:
    - The function retrieves the first `window_mode_entry` from the list of modes associated with the `window_pane`.
    - If a valid `window_mode_entry` is found, it sets the active pane for the window.
    - It then calls the [`window_copy_scroll1`](#window_copy_scroll1) function to perform the actual scrolling operation.
- **Output**:
    - The function does not return a value; it modifies the state of the window pane by scrolling its content.
- **Functions called**:
    - [`window_copy_scroll1`](#window_copy_scroll1)


---
### window_copy_scroll1<!-- {{#callable:window_copy_scroll1}} -->
The `window_copy_scroll1` function updates the view of a window pane based on the position of a scrollbar slider.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `wp`: A pointer to the `window_pane` structure representing the pane being manipulated.
    - `sl_mpos`: An integer representing the position of the slider in the scrollbar.
    - `my`: An unsigned integer representing the current mouse Y position.
    - `scroll_exit`: An integer flag indicating whether to exit the scroll mode when reaching the top.
- **Control Flow**:
    - The function first calculates the new position of the scrollbar slider based on the mouse position.
    - It checks if the mouse position is within the bounds of the scrollbar and adjusts the slider position accordingly.
    - If there are no modes active or if the current offset cannot be retrieved, the function exits early.
    - The new offset is calculated based on the slider's position and the size of the content.
    - The function then calculates the delta between the current offset and the new offset to determine how much to scroll.
    - The pane's view is updated based on the calculated delta while maintaining the cursor's position.
    - If the scroll exit condition is met, the function resets the window pane mode.
    - Finally, it updates the selection and redraws the screen to reflect the changes.
- **Output**:
    - The function does not return a value; it modifies the state of the window pane and updates the display accordingly.
- **Functions called**:
    - [`window_copy_get_current_offset`](#window_copy_get_current_offset)
    - [`window_copy_find_length`](#window_copy_find_length)
    - [`window_copy_cursor_end_of_line`](#window_copy_cursor_end_of_line)
    - [`window_copy_search_marks`](#window_copy_search_marks)
    - [`window_copy_update_selection`](#window_copy_update_selection)
    - [`window_copy_redraw_screen`](#window_copy_redraw_screen)


---
### window_copy_pageup<!-- {{#callable:window_copy_pageup}} -->
The `window_copy_pageup` function scrolls the content of a window pane up by a specified number of lines.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the window pane to be scrolled.
    - `half_page`: An integer indicating whether to scroll half a page (1) or a full page (0).
- **Control Flow**:
    - The function calls [`window_copy_pageup1`](#window_copy_pageup1) with the first mode of the window pane and the `half_page` argument.
    - The [`window_copy_pageup1`](#window_copy_pageup1) function handles the actual scrolling logic based on the provided arguments.
- **Output**:
    - The function does not return a value; it modifies the state of the window pane by scrolling its content up.
- **Functions called**:
    - [`window_copy_pageup1`](#window_copy_pageup1)


---
### window_copy_pageup1<!-- {{#callable:window_copy_pageup1}} -->
The `window_copy_pageup1` function scrolls the content of a window copy mode up by a specified number of lines.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the state of the window copy mode.
    - `half_page`: An integer flag indicating whether to scroll half a page (1) or a full page (0).
- **Control Flow**:
    - Calculate the current offset (`oy`) and the length of the current line (`ox`).
    - If the current cursor position (`cx`) is not equal to `ox`, update the last cursor position.
    - Determine the number of lines to scroll (`n`), either half the screen height or the full height minus two.
    - Check if the new offset after scrolling exceeds the maximum height of the backing screen; if so, adjust the offset and cursor position accordingly.
    - If there is no selection or rectangle flag is not set, check if the cursor needs to be moved to the end of the line.
    - If there are search marks and no timeout, update the search marks.
    - Update the selection and redraw the screen.
- **Output**:
    - The function does not return a value but modifies the state of the window copy mode, including the cursor position and the visible content.
- **Functions called**:
    - [`window_copy_find_length`](#window_copy_find_length)
    - [`window_copy_cursor_end_of_line`](#window_copy_cursor_end_of_line)
    - [`window_copy_search_marks`](#window_copy_search_marks)
    - [`window_copy_update_selection`](#window_copy_update_selection)
    - [`window_copy_redraw_screen`](#window_copy_redraw_screen)


---
### window_copy_pagedown<!-- {{#callable:window_copy_pagedown}} -->
The `window_copy_pagedown` function scrolls the content of a window pane down by a specified number of lines.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the window pane to be scrolled.
    - `half_page`: An integer indicating whether to scroll half a page (1) or a full page (0).
    - `scroll_exit`: An integer indicating whether to exit the copy mode if the scroll reaches the end (1) or not (0).
- **Control Flow**:
    - The function calls [`window_copy_pagedown1`](#window_copy_pagedown1) with the first mode of the window pane, the `half_page` parameter, and the `scroll_exit` parameter.
    - If [`window_copy_pagedown1`](#window_copy_pagedown1) returns a non-zero value, indicating that the scroll reached the end, the function calls `window_pane_reset_mode` to exit the copy mode.
- **Output**:
    - The function does not return a value; it modifies the state of the window pane by scrolling its content down and potentially exiting copy mode.
- **Functions called**:
    - [`window_copy_pagedown1`](#window_copy_pagedown1)


---
### window_copy_pagedown1<!-- {{#callable:window_copy_pagedown1}} -->
The `window_copy_pagedown1` function scrolls the content of a window down by a specified number of lines.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
    - `half_page`: An integer flag indicating whether to scroll half a page (1) or a full page (0).
    - `scroll_exit`: An integer flag indicating whether to exit the copy mode when scrolling reaches the end (1) or not (0).
- **Control Flow**:
    - Calculate the current offset and length of the line at the cursor position.
    - Determine the number of lines to scroll based on the `half_page` flag.
    - Adjust the scroll offset (`oy`) based on the calculated number of lines.
    - If the new offset exceeds the maximum, adjust the cursor position accordingly.
    - If there is no selection or rectangle flag is not set, check if the cursor needs to be moved to the end of the line.
    - If `scroll_exit` is set and the offset is zero, return 1 to indicate exit.
    - If there are search marks, update them based on the current search regex.
    - Update the selection and redraw the screen.
- **Output**:
    - Returns 1 if the scrolling reaches the top of the content and `scroll_exit` is set, otherwise returns 0.
- **Functions called**:
    - [`window_copy_find_length`](#window_copy_find_length)
    - [`window_copy_cursor_end_of_line`](#window_copy_cursor_end_of_line)
    - [`window_copy_search_marks`](#window_copy_search_marks)
    - [`window_copy_update_selection`](#window_copy_update_selection)
    - [`window_copy_redraw_screen`](#window_copy_redraw_screen)


---
### window_copy_previous_paragraph<!-- {{#callable:window_copy_previous_paragraph}} -->
The `window_copy_previous_paragraph` function scrolls the window to the previous paragraph in copy mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
- **Control Flow**:
    - Calculates the current vertical position `oy` based on the backing screen size and current cursor position.
    - Decrements `oy` until it finds a non-empty line, indicating the start of a paragraph.
    - Continues decrementing `oy` until it finds the end of the previous paragraph.
    - Calls [`window_copy_scroll_to`](#window_copy_scroll_to) to update the window's view to the new paragraph position.
- **Output**:
    - The function does not return a value; it modifies the window's view to display the previous paragraph.
- **Functions called**:
    - [`window_copy_find_length`](#window_copy_find_length)
    - [`window_copy_scroll_to`](#window_copy_scroll_to)


---
### window_copy_next_paragraph<!-- {{#callable:window_copy_next_paragraph}} -->
The `window_copy_next_paragraph` function scrolls the window copy mode to the next paragraph.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
- **Control Flow**:
    - Calculates the current vertical position (`oy`) based on the screen height and current cursor position.
    - Determines the maximum vertical position (`maxy`) that can be scrolled to.
    - Iterates to find the next non-empty line by incrementing `oy` until a line with content is found.
    - Continues to increment `oy` until a line with content is found, effectively skipping empty lines.
    - Calculates the length of the found line and scrolls the window to that position.
- **Output**:
    - The function does not return a value; it modifies the state of the window copy mode by scrolling to the next paragraph.
- **Functions called**:
    - [`window_copy_find_length`](#window_copy_find_length)
    - [`window_copy_scroll_to`](#window_copy_scroll_to)


---
### window_copy_get_word<!-- {{#callable:window_copy_get_word}} -->
Retrieves a word from the grid at specified coordinates.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the current window pane.
    - `x`: An unsigned integer representing the x-coordinate in the grid.
    - `y`: An unsigned integer representing the y-coordinate in the grid.
- **Control Flow**:
    - Fetches the first `window_mode_entry` from the modes list of the `window_pane`.
    - Accesses the `window_copy_mode_data` associated with the `window_mode_entry`.
    - Retrieves the `grid` from the backing screen of the `window_copy_mode_data`.
    - Calls the `format_grid_word` function with the grid and adjusted coordinates to get the word.
- **Output**:
    - Returns a pointer to a string containing the word found at the specified coordinates in the grid.


---
### window_copy_get_line<!-- {{#callable:window_copy_get_line}} -->
Retrieves a specific line from the window copy mode's grid.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the current window pane.
    - `y`: An unsigned integer representing the line number to retrieve from the grid.
- **Control Flow**:
    - Fetches the first `window_mode_entry` from the modes list of the `window_pane`.
    - Accesses the `window_copy_mode_data` associated with the `window_mode_entry`.
    - Retrieves the `grid` structure from the `screen` of the `window_copy_mode_data`.
    - Calls the `format_grid_line` function with the grid and the calculated line index to get the line content.
- **Output**:
    - Returns a pointer to a string containing the specified line from the grid.


---
### window_copy_get_hyperlink<!-- {{#callable:window_copy_get_hyperlink}} -->
Retrieves a hyperlink from the specified coordinates in a window pane.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the current window pane.
    - `x`: An unsigned integer representing the x-coordinate in the window pane.
    - `y`: An unsigned integer representing the y-coordinate in the window pane.
- **Control Flow**:
    - The function retrieves the first mode entry from the `window_pane`'s modes list.
    - It accesses the `window_copy_mode_data` associated with the mode entry.
    - It retrieves the grid from the `screen` structure within the `window_copy_mode_data`.
    - The function calls `format_grid_hyperlink` with the grid, adjusted coordinates, and the screen to get the hyperlink.
- **Output**:
    - Returns a pointer to a string containing the hyperlink, or NULL if no hyperlink is found.


---
### window_copy_cursor_hyperlink_cb<!-- {{#callable:window_copy_cursor_hyperlink_cb}} -->
The `window_copy_cursor_hyperlink_cb` function retrieves the hyperlink associated with the current cursor position in a window copy mode.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains formatting information.
- **Control Flow**:
    - The function begins by obtaining the current `window_pane` associated with the provided `format_tree`.
    - It then retrieves the first `window_mode_entry` from the modes of the `window_pane`.
    - Next, it accesses the `window_copy_mode_data` structure from the `window_mode_entry` to get the current grid.
    - Finally, it calls the `format_grid_hyperlink` function with the grid and the current cursor position to retrieve the hyperlink.
- **Output**:
    - The function returns a pointer to the hyperlink associated with the cursor position, or NULL if no hyperlink exists.


---
### window_copy_cursor_word_cb<!-- {{#callable:window_copy_cursor_word_cb}} -->
The `window_copy_cursor_word_cb` function retrieves the word at the current cursor position in a window copy mode.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains formatting information and context.
- **Control Flow**:
    - The function begins by obtaining the `window_pane` associated with the provided `format_tree` using the `format_get_pane` function.
    - It then retrieves the first `window_mode_entry` from the `modes` list of the `window_pane`.
    - Next, it accesses the `window_copy_mode_data` structure from the `window_mode_entry` to get the current cursor position.
    - Finally, it calls the [`window_copy_get_word`](#window_copy_get_word) function with the `window_pane` and the current cursor coordinates to retrieve the word at that position.
- **Output**:
    - The function returns a pointer to a string containing the word found at the cursor's current position.
- **Functions called**:
    - [`window_copy_get_word`](#window_copy_get_word)


---
### window_copy_cursor_line_cb<!-- {{#callable:window_copy_cursor_line_cb}} -->
The `window_copy_cursor_line_cb` function retrieves the content of a specific line from a window pane in copy mode.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains formatting information and context.
- **Control Flow**:
    - The function begins by obtaining the `window_pane` associated with the provided `format_tree` using the `format_get_pane` function.
    - It then retrieves the first `window_mode_entry` from the `modes` list of the `window_pane`.
    - Next, it accesses the `window_copy_mode_data` structure from the `window_mode_entry` to get the current cursor position.
    - Finally, it calls the [`window_copy_get_line`](#window_copy_get_line) function to fetch the line content at the cursor's vertical position and returns it.
- **Output**:
    - The function returns a pointer to a string containing the content of the line at the cursor's current vertical position in the window pane.
- **Functions called**:
    - [`window_copy_get_line`](#window_copy_get_line)


---
### window_copy_search_match_cb<!-- {{#callable:window_copy_search_match_cb}} -->
The `window_copy_search_match_cb` function retrieves the current match at the cursor position in the window copy mode.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains formatting information.
- **Control Flow**:
    - The function starts by obtaining the `window_pane` associated with the provided `format_tree` using the `format_get_pane` function.
    - It then retrieves the first `window_mode_entry` from the list of modes associated with the `window_pane`.
    - Next, it accesses the `window_copy_mode_data` structure from the `window_mode_entry` to get the necessary data for the search.
    - Finally, it calls the [`window_copy_match_at_cursor`](#window_copy_match_at_cursor) function with the `window_copy_mode_data` to find and return the match at the current cursor position.
- **Output**:
    - The function returns a pointer to the match found at the cursor position, or NULL if no match is found.
- **Functions called**:
    - [`window_copy_match_at_cursor`](#window_copy_match_at_cursor)


---
### window_copy_formats<!-- {{#callable:window_copy_formats}} -->
The `window_copy_formats` function populates a `format_tree` with various formatting options related to the window copy mode.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure that contains the state of the window copy mode.
    - `ft`: A pointer to a `format_tree` structure where the formatting options will be added.
- **Control Flow**:
    - Retrieve the `window_copy_mode_data` structure from the `window_mode_entry` to access the current state.
    - Get the height size of the backing screen using `screen_hsize`.
    - Retrieve the grid line corresponding to the current cursor position and extract its timestamp.
    - Add the timestamp of the top line to the format tree.
    - Add the current scroll position and rectangle toggle state to the format tree.
    - Add the current cursor position (x and y coordinates) to the format tree.
    - Check if there is a selection; if so, add the selection start and end coordinates to the format tree.
    - Add the search presence and timeout status to the format tree.
    - If a search count is available, add it to the format tree.
    - Add callback functions for search match, cursor word, cursor line, and cursor hyperlink to the format tree.
- **Output**:
    - The function does not return a value but modifies the provided `format_tree` to include various formatting options related to the window copy mode.


---
### window_copy_get_screen<!-- {{#callable:window_copy_get_screen}} -->
The `window_copy_get_screen` function retrieves the backing screen associated with a given window mode entry.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the context for the window mode.
- **Control Flow**:
    - The function accesses the `data` field of the `wme` structure to get the associated `window_copy_mode_data`.
    - It then returns the `backing` field from the `data` structure, which is a pointer to a `struct screen`.
- **Output**:
    - Returns a pointer to a `struct screen` that represents the backing screen for the window copy mode.


---
### window_copy_size_changed<!-- {{#callable:window_copy_size_changed}} -->
The `window_copy_size_changed` function updates the display of the window copy mode when the size of the window changes.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current state of the window copy mode.
- **Control Flow**:
    - Clears any existing selection and marks in the window copy mode.
    - Starts a screen write context for the current screen associated with the window copy mode.
    - Writes all lines from the window copy mode's screen to the output context.
    - Stops the screen write context after writing the lines.
    - If a search is active and has not timed out, it updates the search marks based on the current search regex.
    - Updates the search coordinates to the current cursor position and the current offset.
- **Output**:
    - The function does not return a value but modifies the state of the window copy mode and the display based on the new size.
- **Functions called**:
    - [`window_copy_clear_selection`](#window_copy_clear_selection)
    - [`window_copy_clear_marks`](#window_copy_clear_marks)
    - [`window_copy_write_lines`](#window_copy_write_lines)
    - [`window_copy_search_marks`](#window_copy_search_marks)


---
### window_copy_resize<!-- {{#callable:window_copy_resize}} -->
The `window_copy_resize` function adjusts the size of the window copy mode screen and updates the cursor position accordingly.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure that contains the state of the window copy mode.
    - `sx`: An unsigned integer representing the new width of the window.
    - `sy`: An unsigned integer representing the new height of the window.
- **Control Flow**:
    - The function begins by retrieving the `screen` and `grid` associated with the `window_mode_entry`.
    - It calls `screen_resize` to adjust the size of the screen to the new dimensions.
    - The current cursor position is calculated based on the existing cursor position and the new dimensions.
    - A check is performed to determine if the grid width has changed, which may require reflowing the grid.
    - The cursor position is updated based on the new dimensions and the current position.
    - The function then updates the `cy` and `oy` values to reflect the new cursor position.
    - Finally, it calls [`window_copy_size_changed`](#window_copy_size_changed) to handle any additional updates needed after resizing, and [`window_copy_redraw_screen`](#window_copy_redraw_screen) to refresh the display.
- **Output**:
    - The function does not return a value but modifies the state of the window copy mode, including the screen size and cursor position.
- **Functions called**:
    - [`window_copy_size_changed`](#window_copy_size_changed)
    - [`window_copy_redraw_screen`](#window_copy_redraw_screen)


---
### window_copy_key_table<!-- {{#callable:window_copy_key_table}} -->
The `window_copy_key_table` function returns the appropriate key table name for the current window mode based on the mode keys setting.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains information about the current window mode.
- **Control Flow**:
    - The function retrieves the `window_pane` from the `window_mode_entry` structure.
    - It checks the value of the 'mode-keys' option for the associated window.
    - If the 'mode-keys' option is set to `MODEKEY_VI`, it returns the string 'copy-mode-vi'.
    - Otherwise, it returns the string 'copy-mode'.
- **Output**:
    - The function outputs a constant string indicating the key table name for the current copy mode, either 'copy-mode-vi' or 'copy-mode'.


---
### window_copy_expand_search_string<!-- {{#callable:window_copy_expand_search_string}} -->
The `window_copy_expand_search_string` function expands a search string based on provided arguments and updates the search string in the window copy mode data.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that contains the state of the window copy command, including the window mode entry and arguments.
- **Control Flow**:
    - The function first retrieves the search string from the command state using `args_string`.
    - If the search string is NULL or empty, the function returns 0, indicating no action was taken.
    - If the 'F' argument is present in the command state, it formats the search string using `format_single`.
    - If the formatted string is empty, it frees the allocated memory and returns 0.
    - If the 'F' argument is not present, it duplicates the original search string using `xstrdup`.
    - Finally, it returns 1 to indicate that the search string was successfully updated.
- **Output**:
    - The function returns 1 if the search string was successfully expanded and updated, or 0 if no valid search string was provided.


---
### window_copy_cmd_append_selection<!-- {{#callable:window_copy_cmd_append_selection}} -->
The `window_copy_cmd_append_selection` function appends the current selection in a window copy mode to the clipboard and clears the selection.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - Checks if the session (`s`) associated with the command state (`cs`) is not NULL.
    - If the session is valid, it calls the [`window_copy_append_selection`](#window_copy_append_selection) function to append the current selection.
    - Calls [`window_copy_clear_selection`](#window_copy_clear_selection) to clear the current selection.
    - Returns `WINDOW_COPY_CMD_REDRAW` to indicate that the screen should be redrawn.
- **Output**:
    - Returns an enumeration value indicating that the window should be redrawn after the selection has been appended and cleared.
- **Functions called**:
    - [`window_copy_append_selection`](#window_copy_append_selection)
    - [`window_copy_clear_selection`](#window_copy_clear_selection)


---
### window_copy_cmd_append_selection_and_cancel<!-- {{#callable:window_copy_cmd_append_selection_and_cancel}} -->
The `window_copy_cmd_append_selection_and_cancel` function appends the current selection to a clipboard and cancels the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - Checks if the session (`s`) associated with the command state (`cs`) is not NULL.
    - If the session is valid, it calls the [`window_copy_append_selection`](#window_copy_append_selection) function to append the current selection.
    - Calls [`window_copy_clear_selection`](#window_copy_clear_selection) to clear the current selection.
    - Returns `WINDOW_COPY_CMD_CANCEL` to indicate that the command has been canceled.
- **Output**:
    - Returns an enumeration value `WINDOW_COPY_CMD_CANCEL`, indicating that the command has been executed and the window copy mode should be canceled.
- **Functions called**:
    - [`window_copy_append_selection`](#window_copy_append_selection)
    - [`window_copy_clear_selection`](#window_copy_clear_selection)


---
### window_copy_cmd_back_to_indentation<!-- {{#callable:window_copy_cmd_back_to_indentation}} -->
The `window_copy_cmd_back_to_indentation` function moves the cursor back to the indentation level of the current line in a window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `struct window_copy_cmd_state` that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `cs` structure.
    - It calls the [`window_copy_cursor_back_to_indentation`](#window_copy_cursor_back_to_indentation) function, passing the `window_mode_entry` to adjust the cursor position.
    - Finally, it returns the action `WINDOW_COPY_CMD_NOTHING` indicating no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that no additional command actions are needed after executing this function.
- **Functions called**:
    - [`window_copy_cursor_back_to_indentation`](#window_copy_cursor_back_to_indentation)


---
### window_copy_cmd_begin_selection<!-- {{#callable:window_copy_cmd_begin_selection}} -->
Begins a selection in window copy mode.
- **Inputs**:
    - `cs`: A pointer to the `window_copy_cmd_state` structure containing the current command state.
- **Control Flow**:
    - Checks if a mouse event is present in the command state.
    - If a mouse event is detected, it calls [`window_copy_start_drag`](#window_copy_start_drag) to initiate a drag operation.
    - If no mouse event is present, it sets the selection flags to indicate character selection mode.
    - Calls [`window_copy_start_selection`](#window_copy_start_selection) to begin the selection process.
    - Returns `WINDOW_COPY_CMD_REDRAW` to indicate that the screen should be redrawn.
- **Output**:
    - Returns an action indicating that the command has been processed, either doing nothing or requiring a redraw of the window.
- **Functions called**:
    - [`window_copy_start_drag`](#window_copy_start_drag)
    - [`window_copy_start_selection`](#window_copy_start_selection)


---
### window_copy_cmd_stop_selection<!-- {{#callable:window_copy_cmd_stop_selection}} -->
The `window_copy_cmd_stop_selection` function stops the current selection in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `struct window_copy_cmd_state` that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the command state.
    - It accesses the `window_copy_mode_data` associated with the `window_mode_entry`.
    - The function sets the `cursordrag` field of the `window_copy_mode_data` to `CURSORDRAG_NONE`, indicating that no cursor dragging is occurring.
    - It also sets the `lineflag` to `LINE_SEL_NONE`, indicating that no line selection is active.
    - The `selflag` is set to `SEL_CHAR`, indicating that character selection mode is active.
    - Finally, the function returns `WINDOW_COPY_CMD_NOTHING`, indicating no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that no further command action is needed after stopping the selection.


---
### window_copy_cmd_bottom_line<!-- {{#callable:window_copy_cmd_bottom_line}} -->
The `window_copy_cmd_bottom_line` function sets the cursor position to the bottom left of the window copy mode screen.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `cs` structure.
    - It accesses the `window_copy_mode_data` from the `window_mode_entry`.
    - The cursor's x-coordinate (`cx`) is set to 0, moving it to the leftmost position.
    - The cursor's y-coordinate (`cy`) is set to the last row of the screen, calculated as the screen height minus one.
    - The function calls [`window_copy_update_selection`](#window_copy_update_selection) to update the selection state.
    - Finally, it returns `WINDOW_COPY_CMD_REDRAW` to indicate that the screen needs to be redrawn.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_REDRAW`, indicating that the screen should be redrawn to reflect the updated cursor position.
- **Functions called**:
    - [`window_copy_update_selection`](#window_copy_update_selection)


---
### window_copy_cmd_cancel<!-- {{#callable:window_copy_cmd_cancel}} -->
The `window_copy_cmd_cancel` function cancels the current window copy command.
- **Inputs**:
    - None
- **Control Flow**:
    - The function does not contain any control flow statements as it directly returns a predefined enum value.
    - It takes an unused parameter of type `struct window_copy_cmd_state *cs`, which is not utilized in the function.
- **Output**:
    - The function returns the enum value `WINDOW_COPY_CMD_CANCEL`, indicating that the copy command has been canceled.


---
### window_copy_cmd_clear_selection<!-- {{#callable:window_copy_cmd_clear_selection}} -->
Clears the current selection in window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `struct window_copy_cmd_state` that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the command state.
    - It calls the [`window_copy_clear_selection`](#window_copy_clear_selection) function to clear the selection.
    - Finally, it returns the action `WINDOW_COPY_CMD_REDRAW` to indicate that the screen should be redrawn.
- **Output**:
    - Returns an enumeration value indicating that the window should be redrawn after clearing the selection.
- **Functions called**:
    - [`window_copy_clear_selection`](#window_copy_clear_selection)


---
### window_copy_do_copy_end_of_line<!-- {{#callable:window_copy_do_copy_end_of_line}} -->
The `window_copy_do_copy_end_of_line` function copies text from the current cursor position to the end of the line in a window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
    - `pipe`: An integer flag indicating whether the output should be piped.
    - `cancel`: An integer flag indicating whether the operation should be canceled.
- **Control Flow**:
    - The function retrieves the current state of the window copy mode and initializes variables for cursor positions and command arguments.
    - If `pipe` is true, it formats the command and prefix based on the provided arguments.
    - The function starts a selection and moves the cursor to the end of the line.
    - If a session is active, it either copies the selection or pipes the output based on the `pipe` flag.
    - If `cancel` is true, it frees allocated memory and returns a cancel command action.
    - Finally, it clears the selection and restores the cursor position before returning a redraw command action.
- **Output**:
    - The function returns an enumeration value indicating the action taken, which can be a redraw command or a cancel command.
- **Functions called**:
    - [`window_copy_start_selection`](#window_copy_start_selection)
    - [`window_copy_cursor_down`](#window_copy_cursor_down)
    - [`window_copy_cursor_end_of_line`](#window_copy_cursor_end_of_line)
    - [`window_copy_copy_pipe`](#window_copy_copy_pipe)
    - [`window_copy_copy_selection`](#window_copy_copy_selection)
    - [`window_copy_clear_selection`](#window_copy_clear_selection)


---
### window_copy_cmd_copy_end_of_line<!-- {{#callable:window_copy_cmd_copy_end_of_line}} -->
The `window_copy_cmd_copy_end_of_line` function copies the text from the current cursor position to the end of the line in a window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function calls [`window_copy_do_copy_end_of_line`](#window_copy_do_copy_end_of_line) with the command state `cs` and two zero values.
    - The [`window_copy_do_copy_end_of_line`](#window_copy_do_copy_end_of_line) function handles the actual copying logic.
- **Output**:
    - Returns an enumeration value of type `window_copy_cmd_action` indicating the result of the copy operation.
- **Functions called**:
    - [`window_copy_do_copy_end_of_line`](#window_copy_do_copy_end_of_line)


---
### window_copy_cmd_copy_end_of_line_and_cancel<!-- {{#callable:window_copy_cmd_copy_end_of_line_and_cancel}} -->
Copies the text from the end of the current line and cancels the selection.
- **Inputs**:
    - `cs`: A pointer to a `struct window_copy_cmd_state` that holds the state of the window copy command.
- **Control Flow**:
    - Calls the [`window_copy_do_copy_end_of_line`](#window_copy_do_copy_end_of_line) function with parameters `cs`, `0`, and `1`.
    - The [`window_copy_do_copy_end_of_line`](#window_copy_do_copy_end_of_line) function handles the actual copying of text from the end of the line and manages the cancellation of the selection.
- **Output**:
    - Returns the result of the [`window_copy_do_copy_end_of_line`](#window_copy_do_copy_end_of_line) function, which is an enumeration value indicating the action taken.
- **Functions called**:
    - [`window_copy_do_copy_end_of_line`](#window_copy_do_copy_end_of_line)


---
### window_copy_cmd_copy_pipe_end_of_line<!-- {{#callable:window_copy_cmd_copy_pipe_end_of_line}} -->
The `window_copy_cmd_copy_pipe_end_of_line` function copies the content from the current cursor position to the end of the line and pipes it to a specified command.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function calls [`window_copy_do_copy_end_of_line`](#window_copy_do_copy_end_of_line) with parameters `cs`, `1`, and `0`.
    - The parameters indicate that the copy operation should be piped and that no cancellation is requested.
- **Output**:
    - The function returns the result of the [`window_copy_do_copy_end_of_line`](#window_copy_do_copy_end_of_line) function, which is an enumeration value indicating the action taken.
- **Functions called**:
    - [`window_copy_do_copy_end_of_line`](#window_copy_do_copy_end_of_line)


---
### window_copy_cmd_copy_pipe_end_of_line_and_cancel<!-- {{#callable:window_copy_cmd_copy_pipe_end_of_line_and_cancel}} -->
This function copies the content from the end of the current line to the clipboard and cancels the selection.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function calls [`window_copy_do_copy_end_of_line`](#window_copy_do_copy_end_of_line) with the `cs` argument and two additional parameters set to 1, indicating that it should copy to the end of the line and cancel the selection.
- **Output**:
    - Returns the result of the [`window_copy_do_copy_end_of_line`](#window_copy_do_copy_end_of_line) function, which is an enumeration value indicating the action taken.
- **Functions called**:
    - [`window_copy_do_copy_end_of_line`](#window_copy_do_copy_end_of_line)


---
### window_copy_do_copy_line<!-- {{#callable:window_copy_do_copy_line}} -->
The `window_copy_do_copy_line` function copies a specified line of text from a window pane to a clipboard or a pipe.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the command.
    - `pipe`: An integer flag indicating whether to copy the line to a pipe (1) or not (0).
    - `cancel`: An integer flag indicating whether to cancel the operation after copying (1) or not (0).
- **Control Flow**:
    - The function retrieves the current state of the window and the command arguments.
    - It checks if the operation is to be piped or copied directly based on the `pipe` argument.
    - It formats the command and prefix based on the provided arguments.
    - The cursor is moved to the start of the line, and a selection is initiated.
    - The cursor is moved down to the specified line and then to the end of that line.
    - If a session is active, it either copies the selection to a pipe or to the clipboard based on the `pipe` argument.
    - If the `cancel` flag is set, it frees allocated memory and returns a cancel action.
    - Finally, it clears the selection and restores the cursor position before returning a redraw action.
- **Output**:
    - The function returns an enumeration value indicating the action taken, either `WINDOW_COPY_CMD_CANCEL` if canceled or `WINDOW_COPY_CMD_REDRAW` if a redraw is needed.
- **Functions called**:
    - [`window_copy_cursor_start_of_line`](#window_copy_cursor_start_of_line)
    - [`window_copy_start_selection`](#window_copy_start_selection)
    - [`window_copy_cursor_down`](#window_copy_cursor_down)
    - [`window_copy_cursor_end_of_line`](#window_copy_cursor_end_of_line)
    - [`window_copy_copy_pipe`](#window_copy_copy_pipe)
    - [`window_copy_copy_selection`](#window_copy_copy_selection)
    - [`window_copy_clear_selection`](#window_copy_clear_selection)


---
### window_copy_cmd_copy_line<!-- {{#callable:window_copy_cmd_copy_line}} -->
The `window_copy_cmd_copy_line` function initiates the process of copying the current line in a window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `struct window_copy_cmd_state` that holds the state of the window copy command.
- **Control Flow**:
    - The function calls [`window_copy_do_copy_line`](#window_copy_do_copy_line) with the command state `cs` and two zero values as parameters.
    - The return value of [`window_copy_do_copy_line`](#window_copy_do_copy_line) is returned as the output of `window_copy_cmd_copy_line`.
- **Output**:
    - Returns an enumeration value of type `enum window_copy_cmd_action` indicating the result of the copy command.
- **Functions called**:
    - [`window_copy_do_copy_line`](#window_copy_do_copy_line)


---
### window_copy_cmd_copy_line_and_cancel<!-- {{#callable:window_copy_cmd_copy_line_and_cancel}} -->
The `window_copy_cmd_copy_line_and_cancel` function copies the current line in the window copy mode and cancels the operation.
- **Inputs**:
    - `cs`: A pointer to a `struct window_copy_cmd_state` that holds the state of the window copy command.
- **Control Flow**:
    - The function calls [`window_copy_do_copy_line`](#window_copy_do_copy_line) with the command state `cs`, a pipe flag set to 0, and a cancel flag set to 1.
    - The return value of [`window_copy_do_copy_line`](#window_copy_do_copy_line) is returned directly as the output of `window_copy_cmd_copy_line_and_cancel`.
- **Output**:
    - Returns an enum value of type `window_copy_cmd_action` indicating the result of the copy operation, which can be used to determine the next action in the command processing.
- **Functions called**:
    - [`window_copy_do_copy_line`](#window_copy_do_copy_line)


---
### window_copy_cmd_copy_pipe_line<!-- {{#callable:window_copy_cmd_copy_pipe_line}} -->
The `window_copy_cmd_copy_pipe_line` function copies the current line from the window copy mode to a specified pipe.
- **Inputs**:
    - `cs`: A pointer to a `struct window_copy_cmd_state` that holds the state of the window copy command.
- **Control Flow**:
    - The function calls [`window_copy_do_copy_line`](#window_copy_do_copy_line) with parameters `cs`, `1`, and `0`.
    - The parameters indicate that it should copy the current line and not cancel the operation.
- **Output**:
    - Returns the result of the [`window_copy_do_copy_line`](#window_copy_do_copy_line) function, which is an enumeration value indicating the action taken.
- **Functions called**:
    - [`window_copy_do_copy_line`](#window_copy_do_copy_line)


---
### window_copy_cmd_copy_pipe_line_and_cancel<!-- {{#callable:window_copy_cmd_copy_pipe_line_and_cancel}} -->
Copies the current line in the window copy mode and cancels the selection.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - Calls the [`window_copy_do_copy_line`](#window_copy_do_copy_line) function with parameters `cs`, `1`, and `1`.
    - The [`window_copy_do_copy_line`](#window_copy_do_copy_line) function handles the actual copying of the line.
- **Output**:
    - Returns the result of the [`window_copy_do_copy_line`](#window_copy_do_copy_line) function, which indicates the action taken (e.g., whether the copy was successful or not).
- **Functions called**:
    - [`window_copy_do_copy_line`](#window_copy_do_copy_line)


---
### window_copy_cmd_copy_selection_no_clear<!-- {{#callable:window_copy_cmd_copy_selection_no_clear}} -->
The `window_copy_cmd_copy_selection_no_clear` function copies the current selection in a window without clearing it.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry`, `client`, `session`, `winlink`, and `window_pane` from the command state.
    - It checks for an optional argument (prefix) and formats it if provided.
    - If a session is active, it calls [`window_copy_copy_selection`](#window_copy_copy_selection) to perform the copy operation with the specified prefix and settings for paste and clipboard.
    - Finally, it frees the allocated prefix memory and returns a status indicating no action is needed.
- **Output**:
    - The function returns `WINDOW_COPY_CMD_NOTHING`, indicating that no further action is required after copying the selection.
- **Functions called**:
    - [`window_copy_copy_selection`](#window_copy_copy_selection)


---
### window_copy_cmd_copy_selection<!-- {{#callable:window_copy_cmd_copy_selection}} -->
The `window_copy_cmd_copy_selection` function copies the current selection in the window copy mode and clears the selection.
- **Inputs**:
    - `cs`: A pointer to a `struct window_copy_cmd_state` that holds the state of the window copy command.
- **Control Flow**:
    - Calls [`window_copy_cmd_copy_selection_no_clear`](#window_copy_cmd_copy_selection_no_clear) to copy the current selection without clearing it.
    - Calls [`window_copy_clear_selection`](#window_copy_clear_selection) to clear the selection in the window copy mode.
    - Returns `WINDOW_COPY_CMD_REDRAW` to indicate that the screen should be redrawn.
- **Output**:
    - Returns an enumeration value indicating that the window copy mode should be redrawn.
- **Functions called**:
    - [`window_copy_cmd_copy_selection_no_clear`](#window_copy_cmd_copy_selection_no_clear)
    - [`window_copy_clear_selection`](#window_copy_clear_selection)


---
### window_copy_cmd_copy_selection_and_cancel<!-- {{#callable:window_copy_cmd_copy_selection_and_cancel}} -->
This function copies the current selection in the window copy mode and cancels the selection.
- **Inputs**:
    - `cs`: A pointer to a `struct window_copy_cmd_state` that holds the state of the window copy command.
- **Control Flow**:
    - Calls [`window_copy_cmd_copy_selection_no_clear`](#window_copy_cmd_copy_selection_no_clear) to copy the current selection without clearing it.
    - Calls [`window_copy_clear_selection`](#window_copy_clear_selection) to clear the selection in the window mode entry.
    - Returns `WINDOW_COPY_CMD_CANCEL` to indicate that the command has been canceled.
- **Output**:
    - Returns an enumeration value `WINDOW_COPY_CMD_CANCEL` indicating that the copy command was executed and the selection has been canceled.
- **Functions called**:
    - [`window_copy_cmd_copy_selection_no_clear`](#window_copy_cmd_copy_selection_no_clear)
    - [`window_copy_clear_selection`](#window_copy_clear_selection)


---
### window_copy_cmd_cursor_down<!-- {{#callable:window_copy_cmd_cursor_down}} -->
The `window_copy_cmd_cursor_down` function moves the cursor down in the window copy mode by a specified number of lines.
- **Inputs**:
    - `cs`: A pointer to a `struct window_copy_cmd_state` that contains the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the command state.
    - It gets the prefix value from the command state, which indicates how many lines to move down.
    - A loop iterates `np` times, calling [`window_copy_cursor_down`](#window_copy_cursor_down) to move the cursor down one line each time.
    - Finally, the function returns `WINDOW_COPY_CMD_NOTHING` to indicate no further action is needed.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that the command has been executed without any additional actions required.
- **Functions called**:
    - [`window_copy_cursor_down`](#window_copy_cursor_down)


---
### window_copy_cmd_cursor_down_and_cancel<!-- {{#callable:window_copy_cmd_cursor_down_and_cancel}} -->
The `window_copy_cmd_cursor_down_and_cancel` function moves the cursor down in copy mode and cancels the operation if the cursor does not move.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the current cursor position `cy` from the `window_copy_mode_data` structure.
    - It enters a loop that decrements the prefix count `np`, moving the cursor down each time using the [`window_copy_cursor_down`](#window_copy_cursor_down) function.
    - After the loop, it checks if the cursor position has changed; if it hasn't and the offset `oy` is zero, it returns a cancel action.
    - If the cursor has moved, it returns a 'do nothing' action.
- **Output**:
    - The function returns an action indicating whether to cancel the operation or do nothing.
- **Functions called**:
    - [`window_copy_cursor_down`](#window_copy_cursor_down)


---
### window_copy_cmd_cursor_left<!-- {{#callable:window_copy_cmd_cursor_left}} -->
Moves the cursor left by a specified number of positions in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `struct window_copy_cmd_state` that holds the current state of the window copy command, including the window mode entry and prefix count.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the command state.
    - It gets the prefix count from the `window_mode_entry`, which indicates how many positions to move the cursor left.
    - A loop iterates from the prefix count down to zero, calling the [`window_copy_cursor_left`](#window_copy_cursor_left) function to move the cursor left for each count.
    - Finally, the function returns a constant indicating no action is taken (WINDOW_COPY_CMD_NOTHING).
- **Output**:
    - Returns a constant value indicating that no further action is required after moving the cursor.
- **Functions called**:
    - [`window_copy_cursor_left`](#window_copy_cursor_left)


---
### window_copy_cmd_cursor_right<!-- {{#callable:window_copy_cmd_cursor_right}} -->
The `window_copy_cmd_cursor_right` function moves the cursor to the right in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `cs` structure.
    - It accesses the `window_copy_mode_data` associated with the `window_mode_entry`.
    - It retrieves the prefix value from the `window_mode_entry`, which indicates how many times to move the cursor.
    - A loop iterates `np` times, calling [`window_copy_cursor_right`](#window_copy_cursor_right) to move the cursor right, considering if a selection is active and if rectangle mode is enabled.
    - Finally, the function returns `WINDOW_COPY_CMD_NOTHING` to indicate no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that the command has been processed without any additional actions required.
- **Functions called**:
    - [`window_copy_cursor_right`](#window_copy_cursor_right)


---
### window_copy_cmd_scroll_to<!-- {{#callable:window_copy_cmd_scroll_to}} -->
Scrolls the visible content of the window copy mode to a specified line position.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the current state of the window copy command.
    - `to`: An unsigned integer representing the target line number to scroll to.
- **Control Flow**:
    - Calculates the difference between the current cursor position and the target line position to determine the scroll direction.
    - If scrolling up is required and the maximum scroll up amount allows it, the function calls [`window_copy_scroll_up`](#window_copy_scroll_up) to scroll the content up.
    - If scrolling down is required and the maximum scroll down amount allows it, the function calls [`window_copy_scroll_down`](#window_copy_scroll_down) to scroll the content down.
    - Updates the selection after scrolling by calling [`window_copy_update_selection`](#window_copy_update_selection).
    - Returns a command action indicating that the window should be redrawn.
- **Output**:
    - Returns a value of type `enum window_copy_cmd_action`, specifically `WINDOW_COPY_CMD_REDRAW`, indicating that the window needs to be redrawn after the scroll operation.
- **Functions called**:
    - [`window_copy_scroll_up`](#window_copy_scroll_up)
    - [`window_copy_scroll_down`](#window_copy_scroll_down)
    - [`window_copy_update_selection`](#window_copy_update_selection)


---
### window_copy_cmd_scroll_bottom<!-- {{#callable:window_copy_cmd_scroll_bottom}} -->
The `window_copy_cmd_scroll_bottom` function scrolls the view in a window copy mode to the bottom of the screen.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the current state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_copy_mode_data` structure from the `window_copy_cmd_state` structure.
    - It calculates the bottom position of the screen by subtracting one from the screen height.
    - It calls the [`window_copy_cmd_scroll_to`](#window_copy_cmd_scroll_to) function with the calculated bottom position to perform the scrolling action.
- **Output**:
    - The function returns the result of the [`window_copy_cmd_scroll_to`](#window_copy_cmd_scroll_to) function, which indicates the action taken (e.g., scrolling to the bottom).
- **Functions called**:
    - [`window_copy_cmd_scroll_to`](#window_copy_cmd_scroll_to)


---
### window_copy_cmd_scroll_middle<!-- {{#callable:window_copy_cmd_scroll_middle}} -->
Scrolls the view in copy mode to the middle of the screen.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - Calculates the middle line index of the screen based on the height of the screen.
    - Calls the [`window_copy_cmd_scroll_to`](#window_copy_cmd_scroll_to) function with the calculated middle index to perform the scrolling action.
- **Output**:
    - Returns the result of the [`window_copy_cmd_scroll_to`](#window_copy_cmd_scroll_to) function, which indicates the action taken during the scrolling.
- **Functions called**:
    - [`window_copy_cmd_scroll_to`](#window_copy_cmd_scroll_to)


---
### window_copy_cmd_scroll_top<!-- {{#callable:window_copy_cmd_scroll_top}} -->
Scrolls the view in the window copy mode to the top of the content.
- **Inputs**:
    - `cs`: A pointer to a `struct window_copy_cmd_state` that holds the state of the window copy command.
- **Control Flow**:
    - Calls the [`window_copy_cmd_scroll_to`](#window_copy_cmd_scroll_to) function with the command state and a position of 0 to scroll to the top.
    - The [`window_copy_cmd_scroll_to`](#window_copy_cmd_scroll_to) function handles the actual scrolling logic.
- **Output**:
    - Returns the result of the [`window_copy_cmd_scroll_to`](#window_copy_cmd_scroll_to) function, which indicates the action taken during the scroll operation.
- **Functions called**:
    - [`window_copy_cmd_scroll_to`](#window_copy_cmd_scroll_to)


---
### window_copy_cmd_cursor_up<!-- {{#callable:window_copy_cmd_cursor_up}} -->
The `window_copy_cmd_cursor_up` function moves the cursor up in the window copy mode by a specified number of lines.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `cs` structure.
    - It accesses the `prefix` value from the `window_mode_entry`, which indicates how many lines to move the cursor up.
    - A loop iterates `np` times, calling the [`window_copy_cursor_up`](#window_copy_cursor_up) function to move the cursor up one line each time.
    - Finally, the function returns `WINDOW_COPY_CMD_NOTHING` to indicate no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that no additional command action is needed after moving the cursor.
- **Functions called**:
    - [`window_copy_cursor_up`](#window_copy_cursor_up)


---
### window_copy_cmd_end_of_line<!-- {{#callable:window_copy_cmd_end_of_line}} -->
The `window_copy_cmd_end_of_line` function moves the cursor to the end of the current line in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the command state.
    - It calls the [`window_copy_cursor_end_of_line`](#window_copy_cursor_end_of_line) function to move the cursor to the end of the line.
    - Finally, it returns the action `WINDOW_COPY_CMD_NOTHING` indicating no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that no additional command processing is needed after moving the cursor.
- **Functions called**:
    - [`window_copy_cursor_end_of_line`](#window_copy_cursor_end_of_line)


---
### window_copy_cmd_halfpage_down<!-- {{#callable:window_copy_cmd_halfpage_down}} -->
The `window_copy_cmd_halfpage_down` function scrolls the window copy mode down by half a page.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the command state.
    - It accesses the `window_copy_mode_data` associated with the `window_mode_entry`.
    - It retrieves the prefix value from the `window_mode_entry`, which indicates how many times to perform the scroll action.
    - A loop iterates `np` times, where `np` is the prefix value.
    - Within the loop, it calls [`window_copy_pagedown1`](#window_copy_pagedown1) to scroll down by one page.
    - If [`window_copy_pagedown1`](#window_copy_pagedown1) returns true, indicating a scroll exit condition, the function returns `WINDOW_COPY_CMD_CANCEL`.
    - If the loop completes without triggering a cancel, the function returns `WINDOW_COPY_CMD_NOTHING`.
- **Output**:
    - The function returns an enumeration value indicating the result of the command, either `WINDOW_COPY_CMD_CANCEL` if the scroll action was canceled or `WINDOW_COPY_CMD_NOTHING` if no action was taken.
- **Functions called**:
    - [`window_copy_pagedown1`](#window_copy_pagedown1)


---
### window_copy_cmd_halfpage_down_and_cancel<!-- {{#callable:window_copy_cmd_halfpage_down_and_cancel}} -->
The `window_copy_cmd_halfpage_down_and_cancel` function scrolls the window copy mode down by half a page and cancels the operation if the scroll reaches the end.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `cs` structure.
    - It gets the prefix value from the `window_mode_entry` which indicates how many times to perform the scroll operation.
    - A loop iterates `np` times, where `np` is the prefix value.
    - Within the loop, it calls [`window_copy_pagedown1`](#window_copy_pagedown1) to attempt to scroll down by one page.
    - If [`window_copy_pagedown1`](#window_copy_pagedown1) returns true, indicating that the scroll reached the end, the function returns `WINDOW_COPY_CMD_CANCEL`.
    - If the loop completes without reaching the end, it returns `WINDOW_COPY_CMD_NOTHING`.
- **Output**:
    - The function returns an enumeration value indicating the result of the command, either `WINDOW_COPY_CMD_CANCEL` if the scroll operation was canceled or `WINDOW_COPY_CMD_NOTHING` if no action was taken.
- **Functions called**:
    - [`window_copy_pagedown1`](#window_copy_pagedown1)


---
### window_copy_cmd_halfpage_up<!-- {{#callable:window_copy_cmd_halfpage_up}} -->
The `window_copy_cmd_halfpage_up` function scrolls the content of a window up by half a page.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `window_copy_cmd_state` structure.
    - It accesses the `prefix` value from the `window_mode_entry`, which indicates how many times to scroll up.
    - A loop iterates `np` times, where `np` is the prefix value, calling [`window_copy_pageup1`](#window_copy_pageup1) each time to scroll up one page.
    - Finally, the function returns `WINDOW_COPY_CMD_NOTHING`, indicating no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that the command has been executed without any additional actions needed.
- **Functions called**:
    - [`window_copy_pageup1`](#window_copy_pageup1)


---
### window_copy_cmd_toggle_position<!-- {{#callable:window_copy_cmd_toggle_position}} -->
Toggles the visibility of the position marker in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `struct window_copy_cmd_state` that holds the state of the window copy command.
- **Control Flow**:
    - Accesses the `window_mode_entry` structure from the `cs` input to get the current window copy mode data.
    - Toggles the `hide_position` boolean in the `window_copy_mode_data` structure to show or hide the position marker.
    - Returns the action `WINDOW_COPY_CMD_REDRAW` to indicate that the screen should be redrawn.
- **Output**:
    - Returns an enum value indicating that the window should be redrawn to reflect the change in position marker visibility.


---
### window_copy_cmd_history_bottom<!-- {{#callable:window_copy_cmd_history_bottom}} -->
The `window_copy_cmd_history_bottom` function moves the cursor to the bottom of the history in a window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - Calculates the current cursor position in the history using the `screen_hsize` and `data` fields.
    - Checks if the selection mode is right-to-left and if the cursor is at the end of the selection, it calls [`window_copy_other_end`](#window_copy_other_end) to switch the selection end.
    - Sets the cursor's y-coordinate to the last line of the screen.
    - Calculates the cursor's x-coordinate using [`window_copy_find_length`](#window_copy_find_length) based on the current line.
    - Resets the offset `oy` to 0.
    - If there are search marks and no timeout, it calls [`window_copy_search_marks`](#window_copy_search_marks) to update the search marks.
    - Calls [`window_copy_update_selection`](#window_copy_update_selection) to refresh the selection state.
    - Returns `WINDOW_COPY_CMD_REDRAW` to indicate that the screen needs to be redrawn.
- **Output**:
    - Returns an enum value `WINDOW_COPY_CMD_REDRAW` indicating that the screen should be redrawn after moving the cursor.
- **Functions called**:
    - [`window_copy_other_end`](#window_copy_other_end)
    - [`window_copy_find_length`](#window_copy_find_length)
    - [`window_copy_search_marks`](#window_copy_search_marks)
    - [`window_copy_update_selection`](#window_copy_update_selection)


---
### window_copy_cmd_history_top<!-- {{#callable:window_copy_cmd_history_top}} -->
The `window_copy_cmd_history_top` function resets the cursor position to the top of the command history in a window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - Calculates the current line position (`oy`) based on the screen height and cursor position.
    - If the selection mode is left-right and the current line matches the selection end, it calls [`window_copy_other_end`](#window_copy_other_end) to adjust the selection.
    - Resets the cursor position to the top of the screen by setting `cy` and `cx` to 0 and `oy` to the height of the backing screen.
    - If there is a search mark and no timeout, it calls [`window_copy_search_marks`](#window_copy_search_marks) to update the search marks.
    - Finally, it calls [`window_copy_update_selection`](#window_copy_update_selection) to refresh the selection and returns a redraw command.
- **Output**:
    - Returns `WINDOW_COPY_CMD_REDRAW`, indicating that the screen should be redrawn to reflect the updated cursor position.
- **Functions called**:
    - [`window_copy_other_end`](#window_copy_other_end)
    - [`window_copy_search_marks`](#window_copy_search_marks)
    - [`window_copy_update_selection`](#window_copy_update_selection)


---
### window_copy_cmd_jump_again<!-- {{#callable:window_copy_cmd_jump_again}} -->
The `window_copy_cmd_jump_again` function moves the cursor in the copy mode of a window based on the last jump type specified.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the current `window_mode_entry` and `window_copy_mode_data` from the command state.
    - It checks the `jumptype` field of the `window_copy_mode_data` to determine the type of jump to perform.
    - Based on the `jumptype`, it executes a loop to call the appropriate jump function ([`window_copy_cursor_jump`](#window_copy_cursor_jump), [`window_copy_cursor_jump_back`](#window_copy_cursor_jump_back), [`window_copy_cursor_jump_to`](#window_copy_cursor_jump_to), or [`window_copy_cursor_jump_to_back`](#window_copy_cursor_jump_to_back)) for the number of times specified by `np` (the prefix argument).
    - After executing the jumps, the function returns `WINDOW_COPY_CMD_NOTHING` to indicate no further action is needed.
- **Output**:
    - The function returns `WINDOW_COPY_CMD_NOTHING`, indicating that no further command action is required after the jumps.
- **Functions called**:
    - [`window_copy_cursor_jump`](#window_copy_cursor_jump)
    - [`window_copy_cursor_jump_back`](#window_copy_cursor_jump_back)
    - [`window_copy_cursor_jump_to`](#window_copy_cursor_jump_to)
    - [`window_copy_cursor_jump_to_back`](#window_copy_cursor_jump_to_back)


---
### window_copy_cmd_jump_reverse<!-- {{#callable:window_copy_cmd_jump_reverse}} -->
The `window_copy_cmd_jump_reverse` function moves the cursor in the reverse direction based on the current jump type set in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that contains the state of the window copy command, including the current window mode entry and other relevant data.
- **Control Flow**:
    - The function retrieves the current jump type from the `data` structure associated with the window mode entry.
    - Based on the jump type, it executes a loop that calls the appropriate cursor jump function ([`window_copy_cursor_jump_back`](#window_copy_cursor_jump_back), [`window_copy_cursor_jump`](#window_copy_cursor_jump), [`window_copy_cursor_jump_to_back`](#window_copy_cursor_jump_to_back), or [`window_copy_cursor_jump_to`](#window_copy_cursor_jump_to)) for the specified number of times (defined by `np`).
    - After executing the jumps, the function returns `WINDOW_COPY_CMD_NOTHING` to indicate that no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that the command has been processed without any additional actions needed.
- **Functions called**:
    - [`window_copy_cursor_jump_back`](#window_copy_cursor_jump_back)
    - [`window_copy_cursor_jump`](#window_copy_cursor_jump)
    - [`window_copy_cursor_jump_to_back`](#window_copy_cursor_jump_to_back)
    - [`window_copy_cursor_jump_to`](#window_copy_cursor_jump_to)


---
### window_copy_cmd_middle_line<!-- {{#callable:window_copy_cmd_middle_line}} -->
The `window_copy_cmd_middle_line` function sets the cursor position to the middle of the visible screen in copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `cs` structure.
    - It accesses the `window_copy_mode_data` from the `window_mode_entry`.
    - The cursor's x-coordinate (`cx`) is set to 0, positioning it at the start of the line.
    - The cursor's y-coordinate (`cy`) is calculated as half the height of the screen minus one, effectively placing it in the middle of the visible area.
    - The function then calls [`window_copy_update_selection`](#window_copy_update_selection) to update the selection state.
    - Finally, it returns `WINDOW_COPY_CMD_REDRAW` to indicate that the screen needs to be redrawn.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_REDRAW`, indicating that the screen should be redrawn to reflect the updated cursor position.
- **Functions called**:
    - [`window_copy_update_selection`](#window_copy_update_selection)


---
### window_copy_cmd_previous_matching_bracket<!-- {{#callable:window_copy_cmd_previous_matching_bracket}} -->
The `window_copy_cmd_previous_matching_bracket` function navigates the cursor to the previous matching bracket in a text buffer.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the current state of the window copy command.
- **Control Flow**:
    - The function iterates based on the prefix count (`np`) provided in the command state.
    - For each iteration, it retrieves the current cursor position and the length of the line at that position.
    - It checks if the current character is a bracket; if not, it attempts to move to the previous character.
    - If a bracket is found, it determines the matching opening bracket and continues to search backward until the matching bracket is found.
    - If the cursor reaches the beginning of the line or the buffer without finding a match, it stops searching.
- **Output**:
    - The function returns `WINDOW_COPY_CMD_NOTHING`, indicating that no specific action was taken beyond cursor navigation.
- **Functions called**:
    - [`window_copy_find_length`](#window_copy_find_length)
    - [`window_copy_cursor_previous_word`](#window_copy_cursor_previous_word)
    - [`window_copy_scroll_to`](#window_copy_scroll_to)


---
### window_copy_cmd_next_matching_bracket<!-- {{#callable:window_copy_cmd_next_matching_bracket}} -->
The `window_copy_cmd_next_matching_bracket` function navigates the cursor to the next matching bracket in a text buffer.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the current state of the window copy command.
- **Control Flow**:
    - The function iterates based on the prefix count (`np`) to determine how many times to attempt to find the next matching bracket.
    - For each iteration, it retrieves the current cursor position and the length of the line at that position.
    - It checks if the current character is a bracket; if not, it attempts to move to the next character or word based on the mode (Emacs or Vi).
    - If a matching opening bracket is found, it counts the nested brackets until the corresponding closing bracket is found.
    - If the cursor moves past the end of the line, it wraps to the next line and continues searching.
- **Output**:
    - The function returns an action indicating that no command was executed (WINDOW_COPY_CMD_NOTHING) after attempting to navigate to the next matching bracket.
- **Functions called**:
    - [`window_copy_find_length`](#window_copy_find_length)
    - [`window_copy_scroll_to`](#window_copy_scroll_to)
    - [`window_copy_cmd_previous_matching_bracket`](#window_copy_cmd_previous_matching_bracket)
    - [`window_copy_cursor_next_word_end`](#window_copy_cursor_next_word_end)


---
### window_copy_cmd_next_paragraph<!-- {{#callable:window_copy_cmd_next_paragraph}} -->
The `window_copy_cmd_next_paragraph` function advances the cursor to the next paragraph in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `struct window_copy_cmd_state` that holds the state of the window copy command, including the current window mode entry.
- **Control Flow**:
    - The function retrieves the current `window_mode_entry` from the command state.
    - It accesses the `prefix` value from the `window_mode_entry`, which indicates how many paragraphs to move.
    - A loop iterates `np` times, calling [`window_copy_next_paragraph`](#window_copy_next_paragraph) to move the cursor to the next paragraph.
    - After the loop, the function returns `WINDOW_COPY_CMD_NOTHING` to indicate no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that the command has been executed without any additional actions needed.
- **Functions called**:
    - [`window_copy_next_paragraph`](#window_copy_next_paragraph)


---
### window_copy_cmd_next_space<!-- {{#callable:window_copy_cmd_next_space}} -->
The `window_copy_cmd_next_space` function moves the cursor to the next space in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `struct window_copy_cmd_state` that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the command state.
    - It accesses the prefix value from the `window_mode_entry` which indicates how many times to move the cursor.
    - A loop iterates `np` times, calling [`window_copy_cursor_next_word`](#window_copy_cursor_next_word) to move the cursor to the next word for each iteration.
    - After the loop completes, the function returns `WINDOW_COPY_CMD_NOTHING` to indicate no further action is required.
- **Output**:
    - The function returns an enum value `WINDOW_COPY_CMD_NOTHING`, indicating that the command has been executed without any additional actions needed.
- **Functions called**:
    - [`window_copy_cursor_next_word`](#window_copy_cursor_next_word)


---
### window_copy_cmd_next_space_end<!-- {{#callable:window_copy_cmd_next_space_end}} -->
The `window_copy_cmd_next_space_end` function moves the cursor to the end of the next space in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `cs` structure.
    - It gets the prefix value from the `window_mode_entry` which indicates how many times to move the cursor.
    - A loop iterates `np` times, calling [`window_copy_cursor_next_word_end`](#window_copy_cursor_next_word_end) to move the cursor to the end of the next word for each iteration.
    - Finally, the function returns `WINDOW_COPY_CMD_NOTHING` to indicate no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that no further command action is needed after executing this function.
- **Functions called**:
    - [`window_copy_cursor_next_word_end`](#window_copy_cursor_next_word_end)


---
### window_copy_cmd_next_word<!-- {{#callable:window_copy_cmd_next_word}} -->
The `window_copy_cmd_next_word` function moves the cursor to the next word in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the prefix value from the `window_mode_entry` structure to determine how many words to move.
    - It fetches the word separators from the options associated with the session.
    - A loop iterates `np` times, calling [`window_copy_cursor_next_word`](#window_copy_cursor_next_word) to move the cursor to the next word for each iteration.
    - Finally, it returns `WINDOW_COPY_CMD_NOTHING` to indicate no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that the command has been executed without any additional actions required.
- **Functions called**:
    - [`window_copy_cursor_next_word`](#window_copy_cursor_next_word)


---
### window_copy_cmd_next_word_end<!-- {{#callable:window_copy_cmd_next_word_end}} -->
The `window_copy_cmd_next_word_end` function moves the cursor to the end of the next word based on specified word separators.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the current state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the command state.
    - It gets the prefix count from the `window_mode_entry` which indicates how many times to move the cursor.
    - It retrieves the word separators from the options associated with the session.
    - A loop iterates `np` times, calling [`window_copy_cursor_next_word_end`](#window_copy_cursor_next_word_end) to move the cursor to the end of the next word for each iteration.
    - Finally, it returns `WINDOW_COPY_CMD_NOTHING` to indicate no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that no further action is needed after executing the command.
- **Functions called**:
    - [`window_copy_cursor_next_word_end`](#window_copy_cursor_next_word_end)


---
### window_copy_cmd_other_end<!-- {{#callable:window_copy_cmd_other_end}} -->
The `window_copy_cmd_other_end` function toggles the selection mode in a window copy command based on the prefix argument.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `window_copy_cmd_state` structure.
    - It sets the selection flag in the `window_copy_mode_data` to `SEL_CHAR`.
    - If the prefix number (`np`) is odd, it calls the [`window_copy_other_end`](#window_copy_other_end) function to toggle the selection end.
    - Finally, it returns `WINDOW_COPY_CMD_NOTHING` to indicate no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that no additional command action is needed.
- **Functions called**:
    - [`window_copy_other_end`](#window_copy_other_end)


---
### window_copy_cmd_page_down<!-- {{#callable:window_copy_cmd_page_down}} -->
The `window_copy_cmd_page_down` function handles the command to scroll down a page in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that contains the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the command state.
    - It then retrieves the `window_copy_mode_data` associated with the `window_mode_entry`.
    - The function uses a loop to call [`window_copy_pagedown1`](#window_copy_pagedown1) for the number of times specified by the prefix in the command state.
    - If [`window_copy_pagedown1`](#window_copy_pagedown1) returns true, indicating that the scroll has reached the end, the function returns `WINDOW_COPY_CMD_CANCEL`.
    - If the loop completes without cancellation, the function returns `WINDOW_COPY_CMD_NOTHING`.
- **Output**:
    - The function returns an enumeration value of type `window_copy_cmd_action`, which indicates whether the command was canceled or if no action was taken.
- **Functions called**:
    - [`window_copy_pagedown1`](#window_copy_pagedown1)


---
### window_copy_cmd_page_down_and_cancel<!-- {{#callable:window_copy_cmd_page_down_and_cancel}} -->
The `window_copy_cmd_page_down_and_cancel` function handles the command to scroll down a page in the window copy mode and cancels the operation if the scroll reaches the end.
- **Inputs**:
    - `cs`: A pointer to a `struct window_copy_cmd_state` that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the prefix value from the `window_mode_entry` structure associated with the command state.
    - It enters a loop that iterates `np` times, where `np` is the prefix value.
    - Within the loop, it calls the [`window_copy_pagedown1`](#window_copy_pagedown1) function to attempt to scroll down the window.
    - If [`window_copy_pagedown1`](#window_copy_pagedown1) returns a non-zero value, indicating that the scroll has reached the end, the function returns `WINDOW_COPY_CMD_CANCEL`.
    - If the loop completes without triggering a cancel, the function returns `WINDOW_COPY_CMD_NOTHING`.
- **Output**:
    - The function returns an enumeration value of type `enum window_copy_cmd_action`, which indicates whether the command was canceled or if no action was taken.
- **Functions called**:
    - [`window_copy_pagedown1`](#window_copy_pagedown1)


---
### window_copy_cmd_page_up<!-- {{#callable:window_copy_cmd_page_up}} -->
The `window_copy_cmd_page_up` function scrolls the content of a window copy mode up by a specified number of pages.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `window_copy_cmd_state` structure.
    - It accesses the `prefix` value from the `window_mode_entry`, which indicates how many pages to scroll up.
    - A loop iterates `np` times, calling the [`window_copy_pageup1`](#window_copy_pageup1) function to scroll up one page each time.
    - After scrolling, the function returns `WINDOW_COPY_CMD_NOTHING` to indicate no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that no additional command action is needed after executing the page up operation.
- **Functions called**:
    - [`window_copy_pageup1`](#window_copy_pageup1)


---
### window_copy_cmd_previous_paragraph<!-- {{#callable:window_copy_cmd_previous_paragraph}} -->
The `window_copy_cmd_previous_paragraph` function navigates the cursor to the previous paragraph in a window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `cs` structure.
    - It gets the prefix value from the `window_mode_entry`, which indicates how many paragraphs to move back.
    - A loop iterates `np` times, where `np` is the prefix value, calling [`window_copy_previous_paragraph`](#window_copy_previous_paragraph) each time to move back one paragraph.
    - After the loop, the function returns `WINDOW_COPY_CMD_NOTHING` to indicate no further action is required.
- **Output**:
    - The function returns an enum value `WINDOW_COPY_CMD_NOTHING`, indicating that the command has been executed without any additional actions needed.
- **Functions called**:
    - [`window_copy_previous_paragraph`](#window_copy_previous_paragraph)


---
### window_copy_cmd_previous_space<!-- {{#callable:window_copy_cmd_previous_space}} -->
Moves the cursor to the previous word in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - Retrieve the `window_mode_entry` from the command state.
    - Get the prefix value from the `window_mode_entry` which indicates how many words to move back.
    - Iterate from the prefix value down to zero, calling [`window_copy_cursor_previous_word`](#window_copy_cursor_previous_word) for each iteration to move the cursor back one word.
    - Return `WINDOW_COPY_CMD_NOTHING` to indicate no further action is needed.
- **Output**:
    - Returns an enum value indicating that no command action is performed after moving the cursor.
- **Functions called**:
    - [`window_copy_cursor_previous_word`](#window_copy_cursor_previous_word)


---
### window_copy_cmd_previous_word<!-- {{#callable:window_copy_cmd_previous_word}} -->
Moves the cursor to the previous word in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - Retrieve the `window_mode_entry` from the command state.
    - Get the prefix value from the `window_mode_entry` which indicates how many words to move back.
    - Fetch the word separators from the options.
    - Iterate `np` times, calling [`window_copy_cursor_previous_word`](#window_copy_cursor_previous_word) to move the cursor back one word each time.
    - Return `WINDOW_COPY_CMD_NOTHING` to indicate no further action is required.
- **Output**:
    - Returns `WINDOW_COPY_CMD_NOTHING`, indicating that the command has been executed without any additional actions required.
- **Functions called**:
    - [`window_copy_cursor_previous_word`](#window_copy_cursor_previous_word)


---
### window_copy_cmd_rectangle_on<!-- {{#callable:window_copy_cmd_rectangle_on}} -->
Sets the rectangle selection mode in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `window_copy_cmd_state` structure.
    - It accesses the `window_copy_mode_data` associated with the `window_mode_entry`.
    - The `lineflag` is set to `LINE_SEL_NONE` to indicate no line selection mode.
    - The [`window_copy_rectangle_set`](#window_copy_rectangle_set) function is called with a value of 1 to enable rectangle selection mode.
    - Finally, the function returns `WINDOW_COPY_CMD_NOTHING` to indicate no further action is required.
- **Output**:
    - Returns `WINDOW_COPY_CMD_NOTHING`, indicating that no further action is needed after setting the rectangle selection mode.
- **Functions called**:
    - [`window_copy_rectangle_set`](#window_copy_rectangle_set)


---
### window_copy_cmd_rectangle_off<!-- {{#callable:window_copy_cmd_rectangle_off}} -->
The `window_copy_cmd_rectangle_off` function disables rectangle selection mode in the window copy command state.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the current state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `window_copy_cmd_state` structure.
    - It accesses the `window_copy_mode_data` associated with the `window_mode_entry`.
    - The `lineflag` attribute of the `window_copy_mode_data` is set to `LINE_SEL_NONE`, indicating that no line selection is active.
    - The [`window_copy_rectangle_set`](#window_copy_rectangle_set) function is called with a value of 0 to disable rectangle selection.
    - Finally, the function returns `WINDOW_COPY_CMD_NOTHING`, indicating no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that no further command action is needed.
- **Functions called**:
    - [`window_copy_rectangle_set`](#window_copy_rectangle_set)


---
### window_copy_cmd_rectangle_toggle<!-- {{#callable:window_copy_cmd_rectangle_toggle}} -->
Toggles the rectangle selection mode in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - Sets the `lineflag` of the `window_copy_mode_data` structure to `LINE_SEL_NONE` to indicate no line selection mode.
    - Calls the [`window_copy_rectangle_set`](#window_copy_rectangle_set) function with the negation of the current `rectflag` to toggle the rectangle selection mode.
    - Returns `WINDOW_COPY_CMD_NOTHING` to indicate no further action is required.
- **Output**:
    - Returns an enum value `WINDOW_COPY_CMD_NOTHING`, indicating that no further command action is needed after toggling the rectangle selection.
- **Functions called**:
    - [`window_copy_rectangle_set`](#window_copy_rectangle_set)


---
### window_copy_cmd_scroll_down<!-- {{#callable:window_copy_cmd_scroll_down}} -->
The `window_copy_cmd_scroll_down` function scrolls the cursor down in the window copy mode by a specified number of lines.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the current state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `cs` structure.
    - It then retrieves the `window_copy_mode_data` from the `window_mode_entry`.
    - The function uses the `prefix` value from the `window_mode_entry` to determine how many lines to scroll down.
    - A loop is executed to call [`window_copy_cursor_down`](#window_copy_cursor_down) for each line specified by the prefix.
    - If the `scroll_exit` flag is set and the offset `oy` is zero, the function returns `WINDOW_COPY_CMD_CANCEL`.
    - If the conditions for cancellation are not met, the function returns `WINDOW_COPY_CMD_NOTHING`.
- **Output**:
    - The function returns an enumeration value indicating the action taken: either `WINDOW_COPY_CMD_CANCEL` if the scrolling reaches the end of the content or `WINDOW_COPY_CMD_NOTHING` if no action is required.
- **Functions called**:
    - [`window_copy_cursor_down`](#window_copy_cursor_down)


---
### window_copy_cmd_scroll_down_and_cancel<!-- {{#callable:window_copy_cmd_scroll_down_and_cancel}} -->
The `window_copy_cmd_scroll_down_and_cancel` function scrolls the cursor down in the window copy mode and cancels the operation if the cursor reaches the top of the content.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the current `window_mode_entry` from the command state.
    - It then retrieves the `window_copy_mode_data` associated with the `window_mode_entry`.
    - The function uses a loop to call [`window_copy_cursor_down`](#window_copy_cursor_down) for the number of times specified by the prefix in the command state.
    - After scrolling down, it checks if the `oy` (offset in the y-direction) is zero, indicating that the cursor is at the top of the content.
    - If `oy` is zero, the function returns `WINDOW_COPY_CMD_CANCEL`, signaling that the operation should be canceled.
    - If `oy` is not zero, it returns `WINDOW_COPY_CMD_NOTHING`, indicating that no action is needed.
- **Output**:
    - The function returns an enumeration value indicating the action taken: either canceling the operation or doing nothing.
- **Functions called**:
    - [`window_copy_cursor_down`](#window_copy_cursor_down)


---
### window_copy_cmd_scroll_up<!-- {{#callable:window_copy_cmd_scroll_up}} -->
The `window_copy_cmd_scroll_up` function scrolls the cursor up in the window copy mode by a specified number of lines.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `window_copy_cmd_state` structure.
    - It accesses the `prefix` value from the `window_mode_entry`, which indicates how many lines to scroll up.
    - A loop iterates `np` times, calling the [`window_copy_cursor_up`](#window_copy_cursor_up) function to move the cursor up one line for each iteration.
    - Finally, the function returns `WINDOW_COPY_CMD_NOTHING` to indicate that no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that the command has been executed without any additional actions needed.
- **Functions called**:
    - [`window_copy_cursor_up`](#window_copy_cursor_up)


---
### window_copy_cmd_search_again<!-- {{#callable:window_copy_cmd_search_again}} -->
The `window_copy_cmd_search_again` function performs a search operation in the copy mode of a terminal window, either upwards or downwards, based on the current search type.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the command, including the current window mode entry and other relevant data.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` and `window_copy_mode_data` from the command state.
    - It checks the `searchtype` in the `window_copy_mode_data` to determine the direction of the search.
    - If the search type is `WINDOW_COPY_SEARCHUP`, it calls [`window_copy_search_up`](#window_copy_search_up) for the specified number of times indicated by `np`.
    - If the search type is `WINDOW_COPY_SEARCHDOWN`, it calls [`window_copy_search_down`](#window_copy_search_down) for the specified number of times indicated by `np`.
    - Finally, it returns `WINDOW_COPY_CMD_NOTHING` to indicate that no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that the command has been executed without any additional actions required.
- **Functions called**:
    - [`window_copy_search_up`](#window_copy_search_up)
    - [`window_copy_search_down`](#window_copy_search_down)


---
### window_copy_cmd_search_reverse<!-- {{#callable:window_copy_cmd_search_reverse}} -->
The `window_copy_cmd_search_reverse` function performs a reverse search in the window copy mode based on the current search type.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the command, including the current window mode entry and other relevant data.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` and `window_copy_mode_data` from the command state.
    - It checks the `searchtype` in the `window_copy_mode_data` to determine the direction of the search.
    - If the search type is `WINDOW_COPY_SEARCHUP`, it calls [`window_copy_search_down`](#window_copy_search_down) for the specified number of times.
    - If the search type is `WINDOW_COPY_SEARCHDOWN`, it calls [`window_copy_search_up`](#window_copy_search_up) for the specified number of times.
- **Output**:
    - The function returns `WINDOW_COPY_CMD_NOTHING`, indicating that no specific action is taken beyond performing the search.
- **Functions called**:
    - [`window_copy_search_down`](#window_copy_search_down)
    - [`window_copy_search_up`](#window_copy_search_up)


---
### window_copy_cmd_select_line<!-- {{#callable:window_copy_cmd_select_line}} -->
The `window_copy_cmd_select_line` function initiates a line selection in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the command.
- **Control Flow**:
    - Sets the selection mode to line selection by updating `data->lineflag` and `data->selflag`.
    - Records the current cursor position as the starting point of the selection.
    - Moves the cursor to the start of the current line.
    - Records the starting selection coordinates.
    - Starts the selection process.
    - Moves the cursor to the end of the current line.
    - Records the end coordinates of the selection.
    - Iterates downwards based on the prefix count to extend the selection to additional lines.
- **Output**:
    - Returns `WINDOW_COPY_CMD_REDRAW` to indicate that the screen should be redrawn to reflect the new selection.
- **Functions called**:
    - [`window_copy_cursor_start_of_line`](#window_copy_cursor_start_of_line)
    - [`window_copy_start_selection`](#window_copy_start_selection)
    - [`window_copy_cursor_end_of_line`](#window_copy_cursor_end_of_line)
    - [`window_copy_find_length`](#window_copy_find_length)
    - [`window_copy_cursor_down`](#window_copy_cursor_down)


---
### window_copy_cmd_select_word<!-- {{#callable:window_copy_cmd_select_word}} -->
The `window_copy_cmd_select_word` function selects a word in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the command.
- **Control Flow**:
    - Sets selection flags for line and rectangle selection modes.
    - Retrieves the word separators from session options.
    - Moves the cursor to the previous word based on the separators.
    - Calculates the current cursor position and the next position.
    - Handles single character words by checking if the next character is whitespace.
    - Updates the cursor position and selection based on the calculated positions.
    - Redraws the lines if the selection is updated.
- **Output**:
    - Returns a command action indicating that the screen needs to be redrawn.
- **Functions called**:
    - [`window_copy_cursor_previous_word`](#window_copy_cursor_previous_word)
    - [`window_copy_start_selection`](#window_copy_start_selection)
    - [`window_copy_find_length`](#window_copy_find_length)
    - [`window_copy_in_set`](#window_copy_in_set)
    - [`window_copy_cursor_next_word_end`](#window_copy_cursor_next_word_end)
    - [`window_copy_update_cursor`](#window_copy_update_cursor)
    - [`window_copy_update_selection`](#window_copy_update_selection)
    - [`window_copy_redraw_lines`](#window_copy_redraw_lines)


---
### window_copy_cmd_set_mark<!-- {{#callable:window_copy_cmd_set_mark}} -->
The `window_copy_cmd_set_mark` function sets a mark at the current cursor position in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_copy_mode_data` structure from the `window_copy_cmd_state` structure.
    - It sets the mark's x-coordinate (`mx`) to the current cursor x-coordinate (`cx`).
    - It sets the mark's y-coordinate (`my`) to the calculated y-coordinate based on the screen height and current cursor position.
    - It updates the `showmark` flag to indicate that the mark should be displayed.
    - Finally, it returns the action `WINDOW_COPY_CMD_REDRAW` to indicate that the screen should be redrawn.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_REDRAW`, indicating that the screen needs to be redrawn to reflect the new mark position.


---
### window_copy_cmd_start_of_line<!-- {{#callable:window_copy_cmd_start_of_line}} -->
The `window_copy_cmd_start_of_line` function moves the cursor to the start of the current line in a window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the command state.
    - It calls the [`window_copy_cursor_start_of_line`](#window_copy_cursor_start_of_line) function to move the cursor to the start of the line.
    - Finally, it returns `WINDOW_COPY_CMD_NOTHING` to indicate no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that no additional command actions are needed after executing this function.
- **Functions called**:
    - [`window_copy_cursor_start_of_line`](#window_copy_cursor_start_of_line)


---
### window_copy_cmd_top_line<!-- {{#callable:window_copy_cmd_top_line}} -->
The `window_copy_cmd_top_line` function resets the cursor position to the top-left corner of the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `cs` structure.
    - It accesses the `window_copy_mode_data` associated with the `window_mode_entry`.
    - The cursor coordinates `cx` and `cy` are set to 0, moving the cursor to the top-left corner.
    - The function calls [`window_copy_update_selection`](#window_copy_update_selection) to update the selection state.
    - Finally, it returns `WINDOW_COPY_CMD_REDRAW` to indicate that the screen should be redrawn.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_REDRAW`, indicating that the window should be redrawn to reflect the updated cursor position.
- **Functions called**:
    - [`window_copy_update_selection`](#window_copy_update_selection)


---
### window_copy_cmd_copy_pipe_no_clear<!-- {{#callable:window_copy_cmd_copy_pipe_no_clear}} -->
The `window_copy_cmd_copy_pipe_no_clear` function copies the selected text to a pipe without clearing the selection.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the command arguments from the `cs` structure.
    - It checks if the second argument (prefix) is provided and formats it using `format_single`.
    - If the session is valid and the first argument (command) is not empty, it formats the command.
    - The function then calls [`window_copy_copy_pipe`](#window_copy_copy_pipe) to perform the actual copying operation.
    - Finally, it frees the allocated memory for the command and prefix before returning.
- **Output**:
    - The function returns an action indicating that no further command is needed (WINDOW_COPY_CMD_NOTHING).
- **Functions called**:
    - [`window_copy_copy_pipe`](#window_copy_copy_pipe)


---
### window_copy_cmd_copy_pipe<!-- {{#callable:window_copy_cmd_copy_pipe}} -->
The `window_copy_cmd_copy_pipe` function copies the current selection to a pipe and clears the selection.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - Calls [`window_copy_cmd_copy_pipe_no_clear`](#window_copy_cmd_copy_pipe_no_clear) to perform the copy operation without clearing the selection.
    - Calls [`window_copy_clear_selection`](#window_copy_clear_selection) to clear the current selection.
    - Returns `WINDOW_COPY_CMD_REDRAW` to indicate that the screen should be redrawn.
- **Output**:
    - Returns an enumeration value indicating that the window should be redrawn after the operation.
- **Functions called**:
    - [`window_copy_cmd_copy_pipe_no_clear`](#window_copy_cmd_copy_pipe_no_clear)
    - [`window_copy_clear_selection`](#window_copy_clear_selection)


---
### window_copy_cmd_copy_pipe_and_cancel<!-- {{#callable:window_copy_cmd_copy_pipe_and_cancel}} -->
This function copies the current selection to a pipe and cancels the selection.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - Calls [`window_copy_cmd_copy_pipe_no_clear`](#window_copy_cmd_copy_pipe_no_clear) to copy the current selection to a pipe without clearing the selection.
    - Calls [`window_copy_clear_selection`](#window_copy_clear_selection) to clear the current selection.
    - Returns `WINDOW_COPY_CMD_CANCEL` to indicate that the command has been canceled.
- **Output**:
    - Returns an enumeration value `WINDOW_COPY_CMD_CANCEL` indicating that the command has been canceled.
- **Functions called**:
    - [`window_copy_cmd_copy_pipe_no_clear`](#window_copy_cmd_copy_pipe_no_clear)
    - [`window_copy_clear_selection`](#window_copy_clear_selection)


---
### window_copy_cmd_pipe_no_clear<!-- {{#callable:window_copy_cmd_pipe_no_clear}} -->
The `window_copy_cmd_pipe_no_clear` function pipes a command output from the current window pane without clearing the selection.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - Checks if the session (`s`) and the first argument (`arg0`) are valid and non-empty.
    - If valid, formats the command string using `format_single` with the provided arguments.
    - Calls [`window_copy_pipe`](#window_copy_pipe) to execute the command and send the output to the specified session.
    - Frees the allocated command string before returning.
- **Output**:
    - Returns an action type indicating that no command action is performed (WINDOW_COPY_CMD_NOTHING).
- **Functions called**:
    - [`window_copy_pipe`](#window_copy_pipe)


---
### window_copy_cmd_pipe<!-- {{#callable:window_copy_cmd_pipe}} -->
The `window_copy_cmd_pipe` function executes a command to pipe the current selection in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - Calls [`window_copy_cmd_pipe_no_clear`](#window_copy_cmd_pipe_no_clear) to execute the piping command without clearing the selection.
    - Calls [`window_copy_clear_selection`](#window_copy_clear_selection) to clear the current selection in the window copy mode.
    - Returns `WINDOW_COPY_CMD_REDRAW` to indicate that the screen should be redrawn.
- **Output**:
    - Returns an enumeration value indicating that the window copy mode should be redrawn after executing the command.
- **Functions called**:
    - [`window_copy_cmd_pipe_no_clear`](#window_copy_cmd_pipe_no_clear)
    - [`window_copy_clear_selection`](#window_copy_clear_selection)


---
### window_copy_cmd_pipe_and_cancel<!-- {{#callable:window_copy_cmd_pipe_and_cancel}} -->
The `window_copy_cmd_pipe_and_cancel` function pipes the current selection to a command and cancels the selection.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - Calls [`window_copy_cmd_pipe_no_clear`](#window_copy_cmd_pipe_no_clear) to execute the piping of the current selection without clearing it.
    - Calls [`window_copy_clear_selection`](#window_copy_clear_selection) to clear the current selection.
    - Returns `WINDOW_COPY_CMD_CANCEL` to indicate that the command has been canceled.
- **Output**:
    - Returns an enumeration value `WINDOW_COPY_CMD_CANCEL` indicating that the command has been canceled.
- **Functions called**:
    - [`window_copy_cmd_pipe_no_clear`](#window_copy_cmd_pipe_no_clear)
    - [`window_copy_clear_selection`](#window_copy_clear_selection)


---
### window_copy_cmd_goto_line<!-- {{#callable:window_copy_cmd_goto_line}} -->
This function moves the cursor to a specified line in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that contains the current state of the window copy command, including the arguments passed to the command.
- **Control Flow**:
    - The function retrieves the first argument from the command state using `args_string`.
    - If the argument is not an empty string, it calls [`window_copy_goto_line`](#window_copy_goto_line) with the current window mode entry and the argument.
    - Finally, it returns `WINDOW_COPY_CMD_NOTHING` to indicate that no further action is required.
- **Output**:
    - The function does not return any specific output value but modifies the cursor position in the window copy mode based on the provided line number.
- **Functions called**:
    - [`window_copy_goto_line`](#window_copy_goto_line)


---
### window_copy_cmd_jump_backward<!-- {{#callable:window_copy_cmd_jump_backward}} -->
The `window_copy_cmd_jump_backward` function moves the cursor backward in the copy mode based on a specified jump character.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that contains the current state of the window copy command.
- **Control Flow**:
    - The function retrieves the jump character from the command state arguments.
    - If the jump character is not an empty string, it sets the jump type to `WINDOW_COPY_JUMPBACKWARD`.
    - It frees any previously allocated jump character memory and assigns the new jump character.
    - The function then calls [`window_copy_cursor_jump_back`](#window_copy_cursor_jump_back) for the number of times specified by the prefix argument.
- **Output**:
    - The function returns `WINDOW_COPY_CMD_NOTHING`, indicating that no specific command action is performed beyond moving the cursor.
- **Functions called**:
    - [`window_copy_cursor_jump_back`](#window_copy_cursor_jump_back)


---
### window_copy_cmd_jump_forward<!-- {{#callable:window_copy_cmd_jump_forward}} -->
The `window_copy_cmd_jump_forward` function moves the cursor forward in the copy mode based on a specified jump character.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` and `window_copy_mode_data` from the command state.
    - It checks if the first argument (jump character) is not an empty string.
    - If the jump character is valid, it sets the jump type to `WINDOW_COPY_JUMPFORWARD`.
    - It frees any previously allocated jump character and assigns the new jump character converted from UTF-8.
    - The cursor is then moved forward in the copy mode for the number of times specified by the prefix argument.
- **Output**:
    - The function returns `WINDOW_COPY_CMD_NOTHING`, indicating that no specific command action is performed after the jump.
- **Functions called**:
    - [`window_copy_cursor_jump`](#window_copy_cursor_jump)


---
### window_copy_cmd_jump_to_backward<!-- {{#callable:window_copy_cmd_jump_to_backward}} -->
The `window_copy_cmd_jump_to_backward` function moves the cursor backward to a specified character in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that contains the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` and `window_copy_mode_data` from the command state.
    - It checks if the first argument (a character) is not empty.
    - If the argument is valid, it sets the jump type to `WINDOW_COPY_JUMPTOBACKWARD` and frees any previously allocated jump character.
    - It then converts the argument to a UTF-8 string and assigns it to `data->jumpchar`.
    - The function then calls [`window_copy_cursor_jump_to_back`](#window_copy_cursor_jump_to_back) for the number of times specified by the prefix argument.
- **Output**:
    - The function returns `WINDOW_COPY_CMD_NOTHING`, indicating that no specific command action is performed after the jump.
- **Functions called**:
    - [`window_copy_cursor_jump_to_back`](#window_copy_cursor_jump_to_back)


---
### window_copy_cmd_jump_to_forward<!-- {{#callable:window_copy_cmd_jump_to_forward}} -->
The `window_copy_cmd_jump_to_forward` function moves the cursor to the next occurrence of a specified character in a copy mode interface.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the current state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` and `window_copy_mode_data` from the command state.
    - It checks if the first argument (a character to jump to) is not empty.
    - If the argument is valid, it sets the jump type to `WINDOW_COPY_JUMPTOFORWARD` and frees any previously allocated jump character.
    - It then converts the argument to a UTF-8 string and assigns it to `jumpchar`.
    - The cursor is moved to the specified character's position in the text for the number of times specified by the prefix argument.
- **Output**:
    - The function returns `WINDOW_COPY_CMD_NOTHING`, indicating that no specific command action is performed beyond moving the cursor.
- **Functions called**:
    - [`window_copy_cursor_jump_to`](#window_copy_cursor_jump_to)


---
### window_copy_cmd_jump_to_mark<!-- {{#callable:window_copy_cmd_jump_to_mark}} -->
The `window_copy_cmd_jump_to_mark` function moves the cursor to a previously set mark in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the current command state, including the window mode entry.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the command state.
    - It calls the [`window_copy_jump_to_mark`](#window_copy_jump_to_mark) function, passing the `window_mode_entry` to move the cursor to the mark.
    - Finally, it returns the action `WINDOW_COPY_CMD_NOTHING` indicating no further action is required.
- **Output**:
    - The function returns an enumeration value `WINDOW_COPY_CMD_NOTHING`, indicating that no further command action is needed after executing the jump to mark.
- **Functions called**:
    - [`window_copy_jump_to_mark`](#window_copy_jump_to_mark)


---
### window_copy_cmd_next_prompt<!-- {{#callable:window_copy_cmd_next_prompt}} -->
The `window_copy_cmd_next_prompt` function updates the cursor position to the next prompt in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `window_copy_cmd_state` structure.
    - It calls the [`window_copy_cursor_prompt`](#window_copy_cursor_prompt) function with the `window_mode_entry`, a direction value of 1, and a boolean indicating if the output should be shown based on the command arguments.
    - Finally, it returns the action `WINDOW_COPY_CMD_NOTHING`.
- **Output**:
    - The function returns an enumeration value indicating that no action is required after executing the command.
- **Functions called**:
    - [`window_copy_cursor_prompt`](#window_copy_cursor_prompt)


---
### window_copy_cmd_previous_prompt<!-- {{#callable:window_copy_cmd_previous_prompt}} -->
The `window_copy_cmd_previous_prompt` function displays the previous prompt in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - The function retrieves the `window_mode_entry` from the `window_copy_cmd_state` structure.
    - It calls the [`window_copy_cursor_prompt`](#window_copy_cursor_prompt) function with the `window_mode_entry`, a value of 0, and a boolean indicating if the 'o' argument is present in the command arguments.
    - Finally, it returns the action `WINDOW_COPY_CMD_NOTHING`.
- **Output**:
    - The function returns an enum value indicating that no action is required after executing the command.
- **Functions called**:
    - [`window_copy_cursor_prompt`](#window_copy_cursor_prompt)


---
### window_copy_cmd_search_backward<!-- {{#callable:window_copy_cmd_search_backward}} -->
The `window_copy_cmd_search_backward` function performs a backward search in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the current command state, including the window mode entry and other relevant data.
- **Control Flow**:
    - The function first calls [`window_copy_expand_search_string`](#window_copy_expand_search_string) to expand the search string from the command state.
    - If the search string is valid (not NULL), it sets the search type to `WINDOW_COPY_SEARCHUP` and marks the search as a regex search.
    - It then iterates `np` times (the prefix count) and calls [`window_copy_search_up`](#window_copy_search_up) to perform the search operation.
- **Output**:
    - The function returns `WINDOW_COPY_CMD_NOTHING`, indicating that no specific action is taken after the search operation.
- **Functions called**:
    - [`window_copy_expand_search_string`](#window_copy_expand_search_string)
    - [`window_copy_search_up`](#window_copy_search_up)


---
### window_copy_cmd_search_backward_text<!-- {{#callable:window_copy_cmd_search_backward_text}} -->
The `window_copy_cmd_search_backward_text` function performs a backward search for a specified text string in a copy mode window.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that contains the state of the command, including the window mode entry and other relevant data.
- **Control Flow**:
    - The function first calls [`window_copy_expand_search_string`](#window_copy_expand_search_string) to expand the search string from the command state.
    - If the search string is valid (not NULL), it sets the search type to `WINDOW_COPY_SEARCHUP` and resets the search regex and timeout values.
    - It then iterates `np` times (the prefix value from the command state) to call [`window_copy_search_up`](#window_copy_search_up), which performs the actual search operation.
- **Output**:
    - The function returns `WINDOW_COPY_CMD_NOTHING`, indicating that no specific action is taken after the search operation.
- **Functions called**:
    - [`window_copy_expand_search_string`](#window_copy_expand_search_string)
    - [`window_copy_search_up`](#window_copy_search_up)


---
### window_copy_cmd_search_forward<!-- {{#callable:window_copy_cmd_search_forward}} -->
The `window_copy_cmd_search_forward` function performs a forward search in the window copy mode.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the command, including the current window mode entry and other relevant data.
- **Control Flow**:
    - The function first calls [`window_copy_expand_search_string`](#window_copy_expand_search_string) to expand the search string from the command state.
    - If the search string is valid (not NULL), it sets the search type to `WINDOW_COPY_SEARCHDOWN` and marks the search as a regex search.
    - It then initializes a loop to perform the search downwards for a number of times specified by the prefix in the command state.
    - For each iteration, it calls [`window_copy_search_down`](#window_copy_search_down) to execute the search.
- **Output**:
    - The function returns `WINDOW_COPY_CMD_NOTHING`, indicating that no specific action is taken after the search is performed.
- **Functions called**:
    - [`window_copy_expand_search_string`](#window_copy_expand_search_string)
    - [`window_copy_search_down`](#window_copy_search_down)


---
### window_copy_cmd_search_forward_text<!-- {{#callable:window_copy_cmd_search_forward_text}} -->
The `window_copy_cmd_search_forward_text` function performs a forward search for a specified text string in a copy mode window.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the command, including the window mode entry and search arguments.
- **Control Flow**:
    - The function first calls [`window_copy_expand_search_string`](#window_copy_expand_search_string) to expand the search string from the command state.
    - If the search string is valid (not NULL), it sets the search type to `WINDOW_COPY_SEARCHDOWN` and resets the search regex and timeout values.
    - It then iterates `np` times (the prefix value from the command state) to call [`window_copy_search_down`](#window_copy_search_down), which performs the actual search operation in the specified direction.
- **Output**:
    - The function returns `WINDOW_COPY_CMD_NOTHING`, indicating that no specific action is taken after the search operation.
- **Functions called**:
    - [`window_copy_expand_search_string`](#window_copy_expand_search_string)
    - [`window_copy_search_down`](#window_copy_search_down)


---
### window_copy_cmd_search_backward_incremental<!-- {{#callable:window_copy_cmd_search_backward_incremental}} -->
The `window_copy_cmd_search_backward_incremental` function performs an incremental backward search in a copy mode interface.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the current command state, including the window mode entry and arguments.
- **Control Flow**:
    - The function starts by initializing local variables and logging the search string.
    - It checks if the search coordinates are uninitialized and sets them to the current cursor position if so.
    - If the search string is empty, it clears any marks and returns a redraw action.
    - Based on the prefix character of the search string, it determines the search direction (up or down) and updates the search string.
    - It then calls the appropriate search function ([`window_copy_search_up`](#window_copy_search_up) or [`window_copy_search_down`](#window_copy_search_down)) based on the determined direction.
    - If the search is unsuccessful, it clears marks and returns a redraw action.
- **Output**:
    - The function returns an action indicating whether to redraw the screen, do nothing, or cancel the command, based on the search results.
- **Functions called**:
    - [`window_copy_clear_marks`](#window_copy_clear_marks)
    - [`window_copy_search_up`](#window_copy_search_up)
    - [`window_copy_search_down`](#window_copy_search_down)


---
### window_copy_cmd_search_forward_incremental<!-- {{#callable:window_copy_cmd_search_forward_incremental}} -->
The `window_copy_cmd_search_forward_incremental` function performs an incremental forward search in a copy mode interface.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the current command state, including the window mode entry and arguments.
- **Control Flow**:
    - The function initializes the search parameters and logs the search string.
    - It checks if the search coordinates are uninitialized and sets them to the current cursor position if so.
    - If the search string is empty, it clears any marks and returns a redraw action.
    - Based on the prefix character of the search string, it determines the search direction (down or up) and updates the search string.
    - It calls the appropriate search function ([`window_copy_search_down`](#window_copy_search_down) or [`window_copy_search_up`](#window_copy_search_up)) to perform the search.
    - If the search fails, it clears marks and returns a redraw action.
- **Output**:
    - Returns an action indicating whether to redraw the screen, do nothing, or perform another action based on the search results.
- **Functions called**:
    - [`window_copy_clear_marks`](#window_copy_clear_marks)
    - [`window_copy_search_down`](#window_copy_search_down)
    - [`window_copy_search_up`](#window_copy_search_up)


---
### window_copy_cmd_refresh_from_pane<!-- {{#callable:window_copy_cmd_refresh_from_pane}} -->
The `window_copy_cmd_refresh_from_pane` function refreshes the copy mode display by updating the backing screen from the current pane.
- **Inputs**:
    - `cs`: A pointer to a `window_copy_cmd_state` structure that holds the state of the window copy command.
- **Control Flow**:
    - Checks if the current mode is view mode; if so, it returns `WINDOW_COPY_CMD_NOTHING` without making changes.
    - Frees the existing backing screen and its associated memory.
    - Clones the current screen from the window pane into the backing screen.
    - Calls [`window_copy_size_changed`](#window_copy_size_changed) to update the size of the window copy mode.
    - Returns `WINDOW_COPY_CMD_REDRAW` to indicate that the display needs to be redrawn.
- **Output**:
    - Returns an enumeration value indicating the action to be taken, specifically `WINDOW_COPY_CMD_REDRAW` to refresh the display.
- **Functions called**:
    - [`window_copy_clone_screen`](#window_copy_clone_screen)
    - [`window_copy_size_changed`](#window_copy_size_changed)


---
### window_copy_command<!-- {{#callable:window_copy_command}} -->
The `window_copy_command` function processes commands in the window copy mode of a terminal multiplexer.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `c`: A pointer to the `client` structure representing the client that initiated the command.
    - `s`: A pointer to the `session` structure representing the current session.
    - `wl`: A pointer to the `winlink` structure representing the current window link.
    - `args`: A pointer to the `args` structure containing the command arguments.
    - `m`: A pointer to the `mouse_event` structure representing mouse event data, if applicable.
- **Control Flow**:
    - The function first checks if there are any command arguments; if not, it returns immediately.
    - It retrieves the command string from the arguments and processes mouse movement if applicable.
    - A command state structure is initialized to hold the current state and context.
    - The function iterates over a predefined command table to find a matching command.
    - If a matching command is found, it parses the arguments and executes the corresponding command function.
    - If the command is not a search command and there are search marks, it handles clearing marks based on the command's clear behavior.
    - Finally, it updates the window mode entry's prefix and redraws the screen if necessary.
- **Output**:
    - The function does not return a value but modifies the state of the window copy mode and may redraw the screen based on the executed command.
- **Functions called**:
    - [`window_copy_move_mouse`](#window_copy_move_mouse)
    - [`args_parse`](tmux.h.driver.md#args_parse)
    - [`window_copy_clear_marks`](#window_copy_clear_marks)
    - [`window_copy_redraw_screen`](#window_copy_redraw_screen)


---
### window_copy_scroll_to<!-- {{#callable:window_copy_scroll_to}} -->
The `window_copy_scroll_to` function adjusts the cursor position in a window copy mode based on specified pixel coordinates.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `px`: An unsigned integer representing the target x-coordinate (pixel) to scroll to.
    - `py`: An unsigned integer representing the target y-coordinate (pixel) to scroll to.
    - `no_redraw`: An integer flag indicating whether to skip the screen redraw after scrolling.
- **Control Flow**:
    - The function first sets the current x-coordinate (`data->cx`) to the input `px` value.
    - It checks if the input `py` is within the visible range of the grid; if so, it calculates the new y-coordinate (`data->cy`).
    - If `py` is outside the visible range, it calculates an offset based on the grid size and adjusts `data->cy` accordingly.
    - The function updates the vertical offset (`data->oy`) based on the calculated position.
    - If `no_redraw` is false and there are search marks present, it calls [`window_copy_search_marks`](#window_copy_search_marks) to update the marks.
    - It then updates the selection and redraws the screen if `no_redraw` is false.
- **Output**:
    - The function does not return a value; it modifies the state of the `window_copy_mode_data` structure and updates the display accordingly.
- **Functions called**:
    - [`window_copy_search_marks`](#window_copy_search_marks)
    - [`window_copy_update_selection`](#window_copy_update_selection)
    - [`window_copy_redraw_screen`](#window_copy_redraw_screen)


---
### window_copy_search_compare<!-- {{#callable:window_copy_search_compare}} -->
Compares two grid cells for equality, with an option for case-insensitive comparison.
- **Inputs**:
    - `gd`: Pointer to the first `grid` structure containing the first cell to compare.
    - `px`: Column index of the cell in the first grid.
    - `py`: Row index of the cell in the first grid.
    - `sgd`: Pointer to the second `grid` structure containing the second cell to compare.
    - `spx`: Column index of the cell in the second grid.
    - `cis`: Integer flag indicating whether the comparison should be case-insensitive (non-zero) or case-sensitive (zero).
- **Control Flow**:
    - Retrieve the cell from the first grid at position (px, py) and store it in `gc`.
    - Retrieve the cell from the second grid at position (spx, 0) and store it in `sgc`.
    - Check if the second cell is a tab character and if the first cell is flagged as a tab; if so, return 1 (indicating a match).
    - Compare the sizes and widths of the two cells; if they differ, return 0 (indicating no match).
    - If case-insensitive comparison is requested and both cells are single characters, compare them after converting the first character to lowercase.
    - If all checks pass, use `memcmp` to compare the data of the two cells and return the result.
- **Output**:
    - Returns 1 if the cells are considered equal, 0 otherwise.


---
### window_copy_search_lr<!-- {{#callable:window_copy_search_lr}} -->
The `window_copy_search_lr` function searches for a pattern in a grid from left to right.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the main grid to search.
    - `sgd`: A pointer to the `grid` structure containing the search pattern.
    - `ppx`: A pointer to an unsigned integer where the starting position of the match will be stored.
    - `py`: An unsigned integer representing the row in the grid where the search starts.
    - `first`: An unsigned integer indicating the starting column index for the search.
    - `last`: An unsigned integer indicating the ending column index for the search.
    - `cis`: An integer flag indicating whether the search should be case insensitive.
- **Control Flow**:
    - The function initializes the end line based on the grid's height and current row.
    - It iterates over the columns from 'first' to 'last'.
    - For each column, it checks each cell in the search grid against the corresponding cell in the main grid.
    - If a cell in the main grid is a tab, it adjusts the padding accordingly.
    - It calls [`window_copy_search_compare`](#window_copy_search_compare) to check if the cells match the search pattern.
    - If a complete match is found, it updates the position pointer and returns 1.
    - If no match is found after checking all columns, it returns 0.
- **Output**:
    - Returns 1 if a match is found, otherwise returns 0.
- **Functions called**:
    - [`window_copy_search_compare`](#window_copy_search_compare)


---
### window_copy_search_rl<!-- {{#callable:window_copy_search_rl}} -->
The `window_copy_search_rl` function searches for a specified pattern in a grid from right to left.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the main grid to search.
    - `sgd`: A pointer to the `grid` structure containing the search pattern.
    - `ppx`: A pointer to a `u_int` variable where the x-coordinate of the found match will be stored.
    - `py`: The y-coordinate in the grid where the search will begin.
    - `first`: The starting index in the grid for the search.
    - `last`: The ending index in the grid for the search.
    - `cis`: An integer flag indicating whether the search should be case insensitive.
- **Control Flow**:
    - The function calculates the end line of the grid based on its height and size.
    - It iterates from the last index to the first index, checking each position in the grid.
    - For each position, it checks each cell in the search grid against the corresponding cell in the main grid.
    - If a match is found for all cells in the search grid, the function updates the output parameter and returns success.
    - If no match is found, the function continues until all positions are checked.
- **Output**:
    - Returns 1 if a match is found, updating `ppx` with the x-coordinate of the match; otherwise returns 0.
- **Functions called**:
    - [`window_copy_search_compare`](#window_copy_search_compare)


---
### window_copy_search_lr_regex<!-- {{#callable:window_copy_search_lr_regex}} -->
The `window_copy_search_lr_regex` function performs a left-to-right regex search in a grid, updating the cursor position if a match is found.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` representing the grid in which the search is performed.
    - `ppx`: A pointer to a `u_int` where the x-coordinate of the found match will be stored.
    - `psx`: A pointer to a `u_int` where the x-coordinate of the end of the match will be stored.
    - `py`: The y-coordinate in the grid where the search starts.
    - `first`: The starting index in the line for the search.
    - `last`: The ending index in the line for the search.
    - `reg`: A pointer to a `regex_t` structure containing the compiled regular expression to search for.
- **Control Flow**:
    - The function first checks if the `first` index is greater than or equal to `last`, returning 0 if true, indicating no search is performed.
    - It sets the regex search flags based on the `first` index.
    - A buffer is allocated and filled with the string representation of the grid starting from the specified line and indices.
    - The function enters a loop to check for matches in the grid lines until the end of the grid is reached or the maximum line length is exceeded.
    - If a match is found using `regexec`, the function calculates the new cursor positions based on the match and updates the output parameters.
    - If no match is found, the function cleans up and returns 0.
- **Output**:
    - The function returns 1 if a match is found and updates the cursor positions; otherwise, it returns 0.
- **Functions called**:
    - [`window_copy_stringify`](#window_copy_stringify)
    - [`window_copy_cstrtocellpos`](#window_copy_cstrtocellpos)


---
### window_copy_search_rl_regex<!-- {{#callable:window_copy_search_rl_regex}} -->
The `window_copy_search_rl_regex` function performs a right-to-left regex search on a grid of text.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` representing the text grid to search.
    - `ppx`: A pointer to a `u_int` where the x-coordinate of the found match will be stored.
    - `psx`: A pointer to a `u_int` where the size of the match will be stored.
    - `py`: The y-coordinate in the grid where the search starts.
    - `first`: The starting index of the search range.
    - `last`: The ending index of the search range.
    - `reg`: A pointer to a `regex_t` structure containing the compiled regular expression.
- **Control Flow**:
    - The function begins by setting the `REG_NOTBOL` flag if the search does not start at the beginning of the line.
    - It allocates a buffer to hold the string representation of the grid lines to be searched.
    - The function then concatenates lines from the grid into the buffer until it reaches the end of the specified range or the maximum line length.
    - It performs a regex search using the `regexec` function to find a match in the buffer.
    - If a match is found, it calculates the position of the match in the grid and updates the output parameters accordingly.
    - If no match is found, it sets the output parameters to zero and returns.
- **Output**:
    - The function returns 1 if a match is found, and 0 if no match is found, while updating the output parameters with the match position and size.
- **Functions called**:
    - [`window_copy_stringify`](#window_copy_stringify)
    - [`window_copy_last_regex`](#window_copy_last_regex)


---
### window_copy_cellstring<!-- {{#callable:window_copy_cellstring}} -->
The `window_copy_cellstring` function retrieves the string representation of a cell in a grid line, handling various cell states and encoding.
- **Inputs**:
    - `gl`: A pointer to a `struct grid_line` representing the grid line from which the cell data is to be retrieved.
    - `px`: An unsigned integer representing the index of the cell within the grid line.
    - `size`: A pointer to a size_t variable where the size of the returned string will be stored.
    - `allocated`: A pointer to an integer where a flag indicating whether the returned string was dynamically allocated will be stored.
- **Control Flow**:
    - Check if the cell index `px` is out of bounds; if so, set size to 1, allocated to 0, and return a space character.
    - Retrieve the cell entry from the grid line using the index `px`.
    - If the cell has padding, set size and allocated to 0 and return NULL.
    - If the cell is not extended, set size to 1, allocated to 0, and return the character data directly.
    - If the cell is a tab, set size to 1, allocated to 0, and return a tab character.
    - Convert the extended data of the cell to UTF-8 format and check if the conversion was successful.
    - If the conversion fails, set size and allocated to 0 and return NULL.
    - Allocate memory for the copy of the UTF-8 data, copy it, and return the pointer to the copied string.
- **Output**:
    - Returns a pointer to a string representing the cell's content, or NULL if the cell is empty or an error occurs. If the string is dynamically allocated, the `allocated` flag is set to 1.


---
### window_copy_last_regex<!-- {{#callable:window_copy_last_regex}} -->
The `window_copy_last_regex` function searches for the last occurrence of a regex pattern in a specified buffer and updates the cursor position accordingly.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` representing the grid where the search is performed.
    - `py`: The y-coordinate in the grid where the search starts.
    - `first`: The starting index in the buffer from which to begin the search.
    - `last`: The ending index in the buffer up to which the search is performed.
    - `len`: The length of the buffer.
    - `ppx`: A pointer to a variable where the x-coordinate of the found match will be stored.
    - `psx`: A pointer to a variable where the size of the found match will be stored.
    - `buf`: A pointer to the buffer containing the text to search.
    - `preg`: A pointer to a compiled regex pattern to search for.
    - `eflags`: Flags that modify the behavior of the regex search.
- **Control Flow**:
    - The function initializes variables to track the found position and the current position in the buffer.
    - It enters a loop that continues as long as the regex matches the substring of the buffer.
    - If a match is found, it converts the matched string to cell position coordinates in the grid.
    - If the found position exceeds the specified limits, the loop breaks.
    - The function updates the length of the remaining buffer and continues searching for the next match.
    - If a valid match is found, it updates the output parameters with the position and size of the match.
    - If no match is found, it sets the output parameters to zero and returns.
- **Output**:
    - The function returns 1 if a match is found and updates `ppx` and `psx` with the position and size of the match, or returns 0 if no match is found.
- **Functions called**:
    - [`window_copy_cstrtocellpos`](#window_copy_cstrtocellpos)


---
### window_copy_stringify<!-- {{#callable:window_copy_stringify}} -->
The `window_copy_stringify` function converts a specified range of cells from a grid line into a dynamically allocated string.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` representing the grid from which the string will be copied.
    - `py`: The row index in the grid from which to start copying.
    - `first`: The starting column index in the specified row to begin copying.
    - `last`: The ending column index in the specified row to stop copying.
    - `buf`: A pointer to a character buffer that will hold the resulting string.
    - `size`: A pointer to an unsigned integer that holds the size of the buffer.
- **Control Flow**:
    - The function begins by determining the required buffer size to hold the resulting string, doubling it until it can accommodate the new size.
    - It reallocates the buffer to the new size using `xrealloc`.
    - The function retrieves the specified line from the grid using `grid_peek_line`.
    - It iterates over the specified range of columns from `first` to `last`, retrieving the string representation of each cell using [`window_copy_cellstring`](#window_copy_cellstring).
    - For each cell, it updates the new size and reallocates the buffer if necessary.
    - The cell's string is then copied into the buffer, either as a single character or as a block of memory, depending on the length of the string.
    - If the cell's string was dynamically allocated, it is freed after copying.
    - Finally, the function null-terminates the resulting string and updates the size parameter before returning the buffer.
- **Output**:
    - The function returns a pointer to the dynamically allocated string that contains the concatenated cell strings from the specified range in the grid.
- **Functions called**:
    - [`window_copy_cellstring`](#window_copy_cellstring)


---
### window_copy_cstrtocellpos<!-- {{#callable:window_copy_cstrtocellpos}} -->
The `window_copy_cstrtocellpos` function maps a C string to a cell position in a grid.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` representing the grid where the string will be searched.
    - `ncells`: An unsigned integer representing the number of cells to be processed.
    - `ppx`: A pointer to an unsigned integer that will hold the calculated x position of the cell.
    - `ppy`: A pointer to an unsigned integer that will hold the calculated y position of the cell.
    - `str`: A pointer to a constant character string that will be searched in the grid.
- **Control Flow**:
    - The function begins by allocating an array of cell data structures to hold the string representations of the grid cells.
    - It populates this array by iterating through the grid cells, retrieving their string representations using [`window_copy_cellstring`](#window_copy_cellstring).
    - Next, it searches for the provided string `str` in the populated cell data, checking for matches character by character.
    - If a match is found, the function calculates the final cell position based on the matched cell index and updates the x and y position pointers.
    - Finally, it frees the allocated cell data before exiting.
- **Output**:
    - The function does not return a value; instead, it updates the x and y position pointers to reflect the cell position corresponding to the input string.
- **Functions called**:
    - [`window_copy_cellstring`](#window_copy_cellstring)


---
### window_copy_move_left<!-- {{#callable:window_copy_move_left}} -->
The `window_copy_move_left` function moves the cursor left in a copy mode interface, handling edge cases for wrapping.
- **Inputs**:
    - `struct screen *s`: Pointer to the `screen` structure representing the current screen.
    - `u_int *fx`: Pointer to the current x-coordinate of the cursor.
    - `u_int *fy`: Pointer to the current y-coordinate of the cursor.
    - `int wrapflag`: Flag indicating whether the cursor should wrap around to the end of the screen when moving left from the first column.
- **Control Flow**:
    - Check if the current x-coordinate (*fx) is 0 (leftmost column).
    - If at the top row (y-coordinate *fy is 0) and wrapflag is set, move the cursor to the last column of the last row.
    - If not at the top row, move the cursor to the last column of the previous row.
    - If not at the leftmost column, simply decrement the x-coordinate (*fx) by 1.
- **Output**:
    - The function does not return a value; it modifies the cursor's position through the pointers provided.


---
### window_copy_move_right<!-- {{#callable:window_copy_move_right}} -->
The `window_copy_move_right` function moves the cursor one position to the right in a window copy mode, handling wrapping to the next line if necessary.
- **Inputs**:
    - `s`: A pointer to a `struct screen` representing the current screen.
    - `fx`: A pointer to an unsigned integer representing the current x-coordinate of the cursor.
    - `fy`: A pointer to an unsigned integer representing the current y-coordinate of the cursor.
    - `wrapflag`: An integer flag indicating whether the cursor should wrap to the next line when it reaches the end of the current line.
- **Control Flow**:
    - Check if the current x-coordinate (`*fx`) is at the rightmost edge of the screen (`screen_size_x(s) - 1`).
    - If at the right edge, check if the current y-coordinate (`*fy`) is at the bottom of the screen (`screen_hsize(s) + screen_size_y(s) - 1`).
    - If at the bottom and `wrapflag` is set, reset the cursor to the top left corner (0, 0).
    - If not at the bottom, reset the x-coordinate to 0 and increment the y-coordinate by 1 to move to the next line.
    - If not at the right edge, simply increment the x-coordinate by 1.
- **Output**:
    - The function does not return a value; it modifies the cursor's position directly through the pointers provided.


---
### window_copy_is_lowercase<!-- {{#callable:window_copy_is_lowercase}} -->
The `window_copy_is_lowercase` function checks if all characters in a given string are lowercase.
- **Inputs**:
    - `ptr`: A pointer to a null-terminated string to be checked for lowercase characters.
- **Control Flow**:
    - The function enters a while loop that continues until the end of the string is reached (indicated by the null terminator).
    - Within the loop, it checks if the current character is not equal to its lowercase equivalent using `tolower`.
    - If a character is found that is not lowercase, the function immediately returns 0 (false).
    - If the loop completes without finding any uppercase characters, the function returns 1 (true).
- **Output**:
    - Returns 1 if all characters in the string are lowercase; otherwise, returns 0.


---
### window_copy_search_back_overlap<!-- {{#callable:window_copy_search_back_overlap}} -->
The `window_copy_search_back_overlap` function performs a backward search for a regex pattern in a grid, handling overlapping matches across wrapped lines.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` representing the grid in which the search is performed.
    - `preg`: A pointer to a compiled regex pattern (`regex_t`) used for searching.
    - `ppx`: A pointer to an unsigned integer representing the current x-coordinate of the cursor.
    - `psx`: A pointer to an unsigned integer representing the width of the match found.
    - `ppy`: A pointer to an unsigned integer representing the current y-coordinate of the cursor.
    - `endline`: An unsigned integer representing the last line to search up to.
- **Control Flow**:
    - The function initializes variables to track the end coordinates of the search and the current cursor position.
    - It enters a loop that continues searching as long as a match is found and the cursor is at the start of a line.
    - Within the loop, it checks if the previous line is wrapped and performs a regex search using [`window_copy_search_rl_regex`](#window_copy_search_rl_regex).
    - If a match is found, it updates the end coordinates and checks if the new end coordinates match the old ones.
    - If they match, it updates the cursor position to the found match.
- **Output**:
    - The function does not return a value but updates the cursor position (`ppx` and `ppy`) to the location of the found match.
- **Functions called**:
    - [`window_copy_search_rl_regex`](#window_copy_search_rl_regex)


---
### window_copy_search_jump<!-- {{#callable:window_copy_search_jump}} -->
The [`window_copy_search_jump`](#window_copy_search_jump) function searches for a specified string or regex pattern in a grid, moving the cursor to the found location.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `gd`: A pointer to the `grid` structure representing the grid where the search is performed.
    - `sgd`: A pointer to the `grid` structure containing the string or regex pattern to search for.
    - `fx`: The starting x-coordinate for the search.
    - `fy`: The starting y-coordinate for the search.
    - `endline`: The last line to search up to.
    - `cis`: An integer flag indicating case-insensitive search.
    - `wrap`: An integer flag indicating whether to wrap around the search.
    - `direction`: An integer indicating the search direction (0 for up, 1 for down).
    - `regex`: An integer flag indicating whether to treat the search string as a regex.
- **Control Flow**:
    - If `regex` is set, the function compiles the regex pattern from `sgd` and prepares for the search.
    - Depending on the `direction`, the function iterates through the lines in the grid either upwards or downwards.
    - For each line, it calls either [`window_copy_search_lr_regex`](#window_copy_search_lr_regex) or [`window_copy_search_lr`](#window_copy_search_lr) to find the pattern.
    - If a match is found, the cursor is moved to the found position using [`window_copy_scroll_to`](#window_copy_scroll_to).
    - If no match is found and `wrap` is set, the search continues from the opposite end of the grid.
- **Output**:
    - Returns 1 if a match is found and the cursor is moved, otherwise returns 0.
- **Functions called**:
    - [`window_copy_stringify`](#window_copy_stringify)
    - [`window_copy_search_lr_regex`](#window_copy_search_lr_regex)
    - [`window_copy_search_lr`](#window_copy_search_lr)
    - [`window_copy_search_rl_regex`](#window_copy_search_rl_regex)
    - [`window_copy_search_back_overlap`](#window_copy_search_back_overlap)
    - [`window_copy_search_rl`](#window_copy_search_rl)
    - [`window_copy_scroll_to`](#window_copy_scroll_to)
    - [`window_copy_search_jump`](#window_copy_search_jump)


---
### window_copy_move_after_search_mark<!-- {{#callable:window_copy_move_after_search_mark}} -->
Moves the cursor to the next search mark after the current position.
- **Inputs**:
    - `data`: A pointer to a `window_copy_mode_data` structure containing the state of the window copy mode.
    - `fx`: A pointer to an unsigned integer representing the current x-coordinate of the cursor.
    - `fy`: A pointer to an unsigned integer representing the current y-coordinate of the cursor.
    - `wrapflag`: An integer flag indicating whether the cursor should wrap around to the other side of the screen when reaching the end.
- **Control Flow**:
    - The function first checks if there is a search mark at the current cursor position using [`window_copy_search_mark_at`](#window_copy_search_mark_at).
    - If a search mark is found, it enters a loop to find the next search mark.
    - Within the loop, it checks if the current search mark is different from the starting search mark.
    - If wrapping is not allowed and the cursor is at the end of the grid, the loop breaks.
    - The cursor is moved to the right using [`window_copy_move_right`](#window_copy_move_right) until a different search mark is found or the end of the grid is reached.
- **Output**:
    - The function does not return a value but updates the cursor position indicated by the pointers `fx` and `fy`.
- **Functions called**:
    - [`window_copy_search_mark_at`](#window_copy_search_mark_at)
    - [`window_copy_move_right`](#window_copy_move_right)


---
### window_copy_search<!-- {{#callable:window_copy_search}} -->
The `window_copy_search` function performs a search for a specified string within a window's copy mode, allowing for both regex and plain text searches.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the current window mode.
    - `direction`: An integer indicating the search direction; 0 for up and 1 for down.
    - `regex`: An integer flag indicating whether the search string should be treated as a regular expression.
- **Control Flow**:
    - Check if the search string is a valid regex; if not, set regex to 0.
    - Set the search direction in the `data` structure.
    - Return early if a timeout has occurred.
    - Determine if the search should be visible only or search all based on previous search parameters.
    - Clear previous search marks if necessary.
    - Duplicate the search string and regex flag into the window pane's search parameters.
    - Initialize the search position based on the current cursor position.
    - Prepare a temporary screen to display the search string.
    - Determine the wrap search option from the window's options.
    - Check if the search string is lowercase for case-insensitive search.
    - Adjust the search starting position based on the direction.
    - Perform the search using the [`window_copy_search_jump`](#window_copy_search_jump) function.
    - If a match is found, update the cursor position and redraw the screen.
    - Free the temporary screen resources before returning the result.
- **Output**:
    - Returns 1 if a match is found, 0 if no match is found, or if the search timed out.
- **Functions called**:
    - [`window_copy_clear_marks`](#window_copy_clear_marks)
    - [`window_copy_is_lowercase`](#window_copy_is_lowercase)
    - [`window_copy_move_after_search_mark`](#window_copy_move_after_search_mark)
    - [`window_copy_move_right`](#window_copy_move_right)
    - [`window_copy_move_left`](#window_copy_move_left)
    - [`window_copy_search_jump`](#window_copy_search_jump)
    - [`window_copy_search_marks`](#window_copy_search_marks)
    - [`window_copy_search_mark_at`](#window_copy_search_mark_at)
    - [`window_copy_redraw_screen`](#window_copy_redraw_screen)


---
### window_copy_visible_lines<!-- {{#callable:window_copy_visible_lines}} -->
The `window_copy_visible_lines` function calculates the start and end indices of visible lines in a window copy mode.
- **Inputs**:
    - `data`: A pointer to a `window_copy_mode_data` structure that contains information about the current state of the window copy mode.
    - `start`: A pointer to an unsigned integer where the starting index of visible lines will be stored.
    - `end`: A pointer to an unsigned integer where the ending index of visible lines will be stored.
- **Control Flow**:
    - The function initializes the `start` variable to the total height of the grid minus the vertical offset (`oy`).
    - It enters a loop that decrements `start` until it finds a line that is not wrapped (indicated by the `GRID_LINE_WRAPPED` flag).
    - Once a non-wrapped line is found, the loop breaks, and the function sets the `end` variable to the total height of the grid minus the vertical offset plus the screen height (`sy`).
- **Output**:
    - The function does not return a value but updates the `start` and `end` pointers to reflect the indices of the visible lines in the window copy mode.


---
### window_copy_search_mark_at<!-- {{#callable:window_copy_search_mark_at}} -->
The `window_copy_search_mark_at` function calculates the linear index of a specific point in a grid based on its pixel coordinates.
- **Inputs**:
    - `data`: A pointer to a `struct window_copy_mode_data` that contains the state and data for the window copy mode.
    - `px`: An unsigned integer representing the x-coordinate (pixel) in the grid.
    - `py`: An unsigned integer representing the y-coordinate (pixel) in the grid.
    - `at`: A pointer to an unsigned integer where the calculated index will be stored.
- **Control Flow**:
    - The function first retrieves the grid from the `data` structure.
    - It checks if the provided y-coordinate `py` is within the valid range of the grid's height.
    - If `py` is less than the starting height or greater than the ending height of the grid, the function returns -1.
    - If the checks pass, it calculates the linear index using the formula based on the grid's size and the provided coordinates.
    - Finally, it stores the calculated index in the variable pointed to by `at` and returns 0.
- **Output**:
    - The function returns 0 on success, indicating that the index was successfully calculated, or -1 if the provided coordinates are out of bounds.


---
### window_copy_clip_width<!-- {{#callable:window_copy_clip_width}} -->
The `window_copy_clip_width` function calculates the effective width of a copy operation based on the available space and the current buffer.
- **Inputs**:
    - `width`: The width of the content to be copied.
    - `b`: The current buffer size.
    - `sx`: The width of the screen.
    - `sy`: The height of the screen.
- **Control Flow**:
    - The function checks if the sum of the buffer size `b` and the content width `width` exceeds the total available space (`sx * sy`).
    - If it does exceed, the function calculates the maximum width that can be accommodated by subtracting the buffer size `b` from the total available space.
    - If it does not exceed, the function simply returns the original `width`.
- **Output**:
    - Returns the effective width for the copy operation, which is either the original width or the maximum width that can fit in the available space.


---
### window_copy_search_mark_match<!-- {{#callable:window_copy_search_mark_match}} -->
The `window_copy_search_mark_match` function marks matches in a search operation within a grid based on specified coordinates.
- **Inputs**:
    - `data`: A pointer to a `window_copy_mode_data` structure containing the state of the window copy mode.
    - `px`: The x-coordinate in the grid where the search starts.
    - `py`: The y-coordinate in the grid where the search starts.
    - `width`: The width of the area to search for matches.
    - `regex`: An integer indicating whether the search should use regular expressions (1) or not (0).
- **Control Flow**:
    - The function first checks if there is a search mark at the specified coordinates using [`window_copy_search_mark_at`](#window_copy_search_mark_at).
    - If a mark is found, it clips the width of the search area using [`window_copy_clip_width`](#window_copy_clip_width).
    - It then iterates over the specified width, checking each cell in the grid.
    - If regex is not used, it retrieves the cell at the current position and checks if it is a tab character, adjusting the width accordingly.
    - If the current position does not have a search mark, it assigns the current search generation to that position.
    - Finally, it increments the search generation for the next match.
- **Output**:
    - The function returns the updated width after marking the matches.
- **Functions called**:
    - [`window_copy_search_mark_at`](#window_copy_search_mark_at)
    - [`window_copy_clip_width`](#window_copy_clip_width)


---
### window_copy_search_marks<!-- {{#callable:window_copy_search_marks}} -->
The `window_copy_search_marks` function searches for specified marks in a window copy mode and highlights them.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `ssp`: A pointer to the `screen` structure that represents the screen to search; if NULL, a temporary screen is created.
    - `regex`: An integer flag indicating whether to treat the search string as a regular expression.
    - `visible_only`: An integer flag indicating whether to search only visible lines.
- **Control Flow**:
    - If `ssp` is NULL, a temporary screen is initialized to display the search string.
    - The function checks if the search string is lowercase to determine case sensitivity for regex.
    - If regex is enabled, the search string is converted to a format suitable for regex matching.
    - The function determines the range of lines to search based on the `visible_only` flag.
    - A loop iterates through the specified lines, searching for matches using either regex or simple string matching.
    - If matches are found, they are marked in the `searchmark` array, and the count of found matches is updated.
    - If the search times out or if the search exceeds the specified limits, the function handles these cases appropriately.
    - Finally, the function cleans up any allocated resources and returns a success status.
- **Output**:
    - The function returns 1 on success, indicating that the search was completed, and updates the `searchmark` array with found matches.
- **Functions called**:
    - [`window_copy_is_lowercase`](#window_copy_is_lowercase)
    - [`window_copy_stringify`](#window_copy_stringify)
    - [`window_copy_visible_lines`](#window_copy_visible_lines)
    - [`window_copy_search_lr_regex`](#window_copy_search_lr_regex)
    - [`window_copy_search_lr`](#window_copy_search_lr)
    - [`window_copy_search_mark_match`](#window_copy_search_mark_match)
    - [`window_copy_clear_marks`](#window_copy_clear_marks)


---
### window_copy_clear_marks<!-- {{#callable:window_copy_clear_marks}} -->
The `window_copy_clear_marks` function clears the search marks in the window copy mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the state of the window copy mode.
- **Control Flow**:
    - The function retrieves the `window_copy_mode_data` structure from the `window_mode_entry`.
    - It frees the memory allocated for the `searchmark` field in the `window_copy_mode_data` structure.
    - It sets the `searchmark` pointer to NULL to indicate that there are no active search marks.
- **Output**:
    - The function does not return a value; it modifies the state of the `window_copy_mode_data` by clearing the search marks.


---
### window_copy_search_up<!-- {{#callable:window_copy_search_up}} -->
The `window_copy_search_up` function initiates a search for text in the copy mode of a window, moving upwards through the content.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode.
    - `regex`: An integer flag indicating whether the search should treat the search string as a regular expression.
- **Control Flow**:
    - Calls the [`window_copy_search`](#window_copy_search) function with parameters to search upwards (direction set to 0) and passes the regex flag.
    - The [`window_copy_search`](#window_copy_search) function handles the actual searching logic based on the provided parameters.
- **Output**:
    - Returns the result of the search operation, which is an integer indicating success or failure of the search.
- **Functions called**:
    - [`window_copy_search`](#window_copy_search)


---
### window_copy_search_down<!-- {{#callable:window_copy_search_down}} -->
The `window_copy_search_down` function initiates a downward search in the copy mode of a window.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the state of the window mode.
    - `regex`: An integer indicating whether the search should use regular expressions.
- **Control Flow**:
    - The function calls [`window_copy_search`](#window_copy_search) with the `wme` pointer, a direction value of 1 (indicating downward search), and the `regex` flag.
    - The return value of [`window_copy_search`](#window_copy_search) is directly returned as the output of `window_copy_search_down`.
- **Output**:
    - Returns an integer indicating the result of the search operation, as defined by the [`window_copy_search`](#window_copy_search) function.
- **Functions called**:
    - [`window_copy_search`](#window_copy_search)


---
### window_copy_goto_line<!-- {{#callable:window_copy_goto_line}} -->
The `window_copy_goto_line` function updates the current line position in the window copy mode based on a specified line number.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
    - `linestr`: A string representing the line number to which the window should scroll.
- **Control Flow**:
    - The function converts the `linestr` input to an integer line number using `strtonum`, with error handling for invalid input.
    - If the line number is out of bounds (negative or greater than the total number of lines), it is adjusted to the maximum valid line number.
    - The function updates the vertical offset (`oy`) of the window copy mode data to the new line number.
    - It then calls [`window_copy_update_selection`](#window_copy_update_selection) to refresh the selection state and [`window_copy_redraw_screen`](#window_copy_redraw_screen) to redraw the window with the updated line position.
- **Output**:
    - The function does not return a value; it modifies the state of the window copy mode directly.
- **Functions called**:
    - [`window_copy_update_selection`](#window_copy_update_selection)
    - [`window_copy_redraw_screen`](#window_copy_redraw_screen)


---
### window_copy_match_start_end<!-- {{#callable:window_copy_match_start_end}} -->
The `window_copy_match_start_end` function identifies the start and end positions of a contiguous block of matching characters in a grid.
- **Inputs**:
    - `data`: A pointer to a `window_copy_mode_data` structure that contains the grid and search mark information.
    - `at`: An unsigned integer representing the current position in the search mark array.
    - `start`: A pointer to an unsigned integer where the start position of the match will be stored.
    - `end`: A pointer to an unsigned integer where the end position of the match will be stored.
- **Control Flow**:
    - The function initializes `start` and `end` to the position `at`.
    - It then decrements `start` while the character at `start` matches the character at `at` and `start` is not zero.
    - If the character at `start` does not match, it increments `start` back to the last matching position.
    - Next, it increments `end` while the character at `end` matches the character at `at` and `end` is not the last position in the grid.
    - If the character at `end` does not match, it decrements `end` back to the last matching position.
- **Output**:
    - The function does not return a value but updates the `start` and `end` pointers to reflect the positions of the first and last matching characters in the grid.


---
### window_copy_match_at_cursor<!-- {{#callable:window_copy_match_at_cursor}} -->
The `window_copy_match_at_cursor` function retrieves the text matching the current cursor position in a window copy mode.
- **Inputs**:
    - `data`: A pointer to a `window_copy_mode_data` structure that contains the state and data for the window copy mode.
- **Control Flow**:
    - Checks if the `searchmark` is NULL; if so, returns NULL.
    - Calculates the current cursor position in the grid based on the screen size and offsets.
    - Attempts to find the position of the search mark at the cursor; if not found, returns NULL.
    - If the search mark at the cursor is zero, it checks the previous position; if that is also zero, returns NULL.
    - Calls [`window_copy_match_start_end`](#window_copy_match_start_end) to determine the start and end positions of the match.
    - Iterates from the start to the end position, retrieving each cell in the grid.
    - Handles tab characters and padding appropriately while building the output buffer.
    - Finalizes the buffer by adding a null terminator and returns it.
- **Output**:
    - Returns a dynamically allocated string containing the matched text, or NULL if no match is found.
- **Functions called**:
    - [`window_copy_search_mark_at`](#window_copy_search_mark_at)
    - [`window_copy_match_start_end`](#window_copy_match_start_end)


---
### window_copy_update_style<!-- {{#callable:window_copy_update_style}} -->
The `window_copy_update_style` function updates the visual style of a specific cell in a window copy mode based on various conditions.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure that contains the current window pane and mode data.
    - `fx`: The x-coordinate of the cell to be updated.
    - `fy`: The y-coordinate of the cell to be updated.
    - `gc`: A pointer to a `grid_cell` structure that will be modified to reflect the new style.
    - `mgc`: A pointer to a `grid_cell` structure representing the default style for the cell.
    - `cgc`: A pointer to a `grid_cell` structure representing the style for the current match.
    - `mkgc`: A pointer to a `grid_cell` structure representing the style for the mark.
- **Control Flow**:
    - Check if the mark should be shown and if the current y-coordinate matches the mark's y-coordinate.
    - If so, update the attributes and colors of the cell based on whether it is the mark's position.
    - If there is no search mark, exit the function.
    - Check if the current cell matches a search mark and retrieve its index.
    - If the mark is found, determine the cursor's position and check if it matches the current mark.
    - If a match is found, update the cell's style to reflect the current match.
    - If no match is found, reset the cell's style to the default.
- **Output**:
    - The function modifies the `gc` parameter to reflect the updated style based on the conditions checked, but does not return a value.
- **Functions called**:
    - [`window_copy_search_mark_at`](#window_copy_search_mark_at)
    - [`window_copy_match_start_end`](#window_copy_match_start_end)


---
### window_copy_write_one<!-- {{#callable:window_copy_write_one}} -->
The `window_copy_write_one` function writes a specified number of cells from a grid to a screen context, applying styles based on the provided grid cell parameters.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure that contains the state of the window copy mode.
    - `ctx`: A pointer to a `screen_write_ctx` structure that represents the context for writing to the screen.
    - `py`: The y-coordinate on the screen where the writing should start.
    - `fy`: The y-coordinate in the grid from which to read the cells.
    - `nx`: The number of cells to write from the grid.
    - `mgc`: A pointer to a `grid_cell` structure that defines the style for the match cell.
    - `cgc`: A pointer to a `grid_cell` structure that defines the style for the current cell.
    - `mkgc`: A pointer to a `grid_cell` structure that defines the style for the mark cell.
- **Control Flow**:
    - The function begins by moving the cursor in the `screen_write_ctx` to the specified position (0, py).
    - It then enters a loop that iterates from 0 to nx, retrieving each cell from the grid at the specified x-coordinate (fx) and y-coordinate (fy).
    - For each cell, it checks if the cell's width plus its current x-coordinate does not exceed nx.
    - If the condition is met, it calls [`window_copy_update_style`](#window_copy_update_style) to apply the appropriate styles to the cell based on the provided grid cell parameters.
    - Finally, it writes the styled cell to the screen context using `screen_write_cell`.
- **Output**:
    - The function does not return a value; instead, it directly modifies the screen context by writing the specified cells from the grid to the screen.
- **Functions called**:
    - [`window_copy_update_style`](#window_copy_update_style)


---
### window_copy_get_current_offset<!-- {{#callable:window_copy_get_current_offset}} -->
The `window_copy_get_current_offset` function retrieves the current offset and size of the visible content in a window pane's copy mode.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the window pane from which to get the current offset.
    - `offset`: A pointer to a `u_int` variable where the current offset will be stored.
    - `size`: A pointer to a `u_int` variable where the size of the visible content will be stored.
- **Control Flow**:
    - The function first retrieves the current window mode entry from the `window_pane` structure.
    - It checks if the `data` field of the `window_copy_mode_data` structure is NULL; if it is, the function returns 0.
    - The height of the screen is calculated using the `screen_hsize` function.
    - The current offset is calculated as the height of the screen minus the vertical offset (`oy`) stored in the `data` structure.
    - The size is set to the height of the screen.
    - Finally, the function returns 1 to indicate success.
- **Output**:
    - The function outputs a success status (1) if the operation is successful, or 0 if the `data` field is NULL.


---
### window_copy_write_line<!-- {{#callable:window_copy_write_line}} -->
The `window_copy_write_line` function writes a single line of text to the window copy mode screen.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure that contains the state of the window copy mode.
    - `ctx`: A pointer to the `screen_write_ctx` structure used for writing to the screen.
    - `py`: An unsigned integer representing the line number to be written.
- **Control Flow**:
    - The function retrieves the current window pane and its associated data.
    - It applies various styles to grid cells based on options set in the window pane.
    - It calls [`window_copy_write_one`](#window_copy_write_one) to write the specified line to the screen.
    - If the line being written is the first line and certain conditions are met, it formats and displays the position indicator.
    - If the line being written is the current cursor line, it places a '$' character at the end of the line.
- **Output**:
    - The function does not return a value; it directly modifies the screen output by writing the specified line.
- **Functions called**:
    - [`window_copy_write_one`](#window_copy_write_one)


---
### window_copy_write_lines<!-- {{#callable:window_copy_write_lines}} -->
The `window_copy_write_lines` function writes a specified number of lines from a window copy mode entry to a screen write context.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `ctx`: A pointer to the `screen_write_ctx` structure used for writing to the screen.
    - `py`: An unsigned integer representing the starting line index from which to write.
    - `ny`: An unsigned integer representing the number of lines to write.
- **Control Flow**:
    - The function initializes a loop that iterates from the starting line index `py` to `py + ny`.
    - Within the loop, it calls the [`window_copy_write_line`](#window_copy_write_line) function for each line index to write the corresponding line to the screen.
- **Output**:
    - The function does not return a value; it directly writes lines to the specified screen context.
- **Functions called**:
    - [`window_copy_write_line`](#window_copy_write_line)


---
### window_copy_redraw_selection<!-- {{#callable:window_copy_redraw_selection}} -->
The `window_copy_redraw_selection` function updates the visual representation of the selected text in a copy mode window.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the state of the window in copy mode.
    - `old_y`: An unsigned integer representing the previous vertical position of the cursor before the redraw.
- **Control Flow**:
    - The function retrieves the current cursor position from the `data` structure associated with `wme`.
    - It determines the start and end positions for the redraw based on the current and old cursor positions.
    - If the selection mode is set to word selection (`SEL_WORD`), it adjusts the end position to include the next line if necessary.
    - Finally, it calls [`window_copy_redraw_lines`](#window_copy_redraw_lines) to update the visual representation of the selected lines.
- **Output**:
    - The function does not return a value; it modifies the display of the selected text in the window.
- **Functions called**:
    - [`window_copy_redraw_lines`](#window_copy_redraw_lines)


---
### window_copy_redraw_lines<!-- {{#callable:window_copy_redraw_lines}} -->
The `window_copy_redraw_lines` function redraws a specified range of lines in a window copy mode.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure that contains the state of the window copy mode.
    - `py`: An unsigned integer representing the starting line number to redraw.
    - `ny`: An unsigned integer representing the number of lines to redraw starting from `py`.
- **Control Flow**:
    - The function begins by initializing a `screen_write_ctx` structure to manage the screen writing context.
    - It then enters a loop that iterates from the starting line `py` to `py + ny`, calling the [`window_copy_write_line`](#window_copy_write_line) function for each line to be redrawn.
    - After writing the lines, it moves the cursor to the current position specified by the `data` structure in `wme`.
    - Finally, it stops the screen writing context.
- **Output**:
    - The function does not return a value; it directly modifies the screen output by redrawing the specified lines.
- **Functions called**:
    - [`window_copy_write_line`](#window_copy_write_line)


---
### window_copy_redraw_screen<!-- {{#callable:window_copy_redraw_screen}} -->
Redraws the entire screen for the window copy mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the state of the window copy mode.
- **Control Flow**:
    - Retrieves the `window_copy_mode_data` structure from the `window_mode_entry`.
    - Calls the [`window_copy_redraw_lines`](#window_copy_redraw_lines) function to redraw all lines from the top of the screen to the bottom.
- **Output**:
    - This function does not return a value; it performs a redraw operation on the screen.
- **Functions called**:
    - [`window_copy_redraw_lines`](#window_copy_redraw_lines)


---
### window_copy_synchronize_cursor_end<!-- {{#callable:window_copy_synchronize_cursor_end}} -->
Synchronizes the cursor position in a window copy mode based on the selection type.
- **Inputs**:
    - `wme`: Pointer to the `window_mode_entry` structure representing the current window mode.
    - `begin`: An integer indicating whether to reset the selection start position.
    - `no_reset`: An integer flag indicating whether to skip resetting the selection end position.
- **Control Flow**:
    - Retrieve the current cursor position from the `window_copy_mode_data` structure.
    - Check the selection type (`selflag`) to determine how to synchronize the cursor.
    - For `SEL_WORD`, check if the selection is right-to-left or left-to-right and adjust the cursor accordingly.
    - For `SEL_LINE`, similarly check the direction and adjust the cursor position.
    - If `no_reset` is not set, reset the selection start or end positions based on the selection direction.
    - Update the selection coordinates based on the cursor position.
- **Output**:
    - The function does not return a value but updates the selection coordinates in the `window_copy_mode_data` structure.
- **Functions called**:
    - [`window_copy_cursor_previous_word_pos`](#window_copy_cursor_previous_word_pos)
    - [`window_copy_find_length`](#window_copy_find_length)
    - [`window_copy_in_set`](#window_copy_in_set)
    - [`window_copy_cursor_next_word_end_pos`](#window_copy_cursor_next_word_end_pos)


---
### window_copy_synchronize_cursor<!-- {{#callable:window_copy_synchronize_cursor}} -->
The `window_copy_synchronize_cursor` function synchronizes the cursor position in a window copy mode based on the current drag state.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure that contains the state of the window copy mode.
    - `no_reset`: An integer flag indicating whether to reset the cursor position.
- **Control Flow**:
    - The function retrieves the `window_copy_mode_data` structure from the `window_mode_entry`.
    - It checks the `cursordrag` state of the `window_copy_mode_data`.
    - If `cursordrag` is `CURSORDRAG_ENDSEL`, it calls [`window_copy_synchronize_cursor_end`](#window_copy_synchronize_cursor_end) with parameters to synchronize the cursor to the end of the selection.
    - If `cursordrag` is `CURSORDRAG_SEL`, it calls [`window_copy_synchronize_cursor_end`](#window_copy_synchronize_cursor_end) with parameters to synchronize the cursor to the start of the selection.
    - If `cursordrag` is `CURSORDRAG_NONE`, no action is taken.
- **Output**:
    - The function does not return a value; it modifies the cursor position based on the current drag state.
- **Functions called**:
    - [`window_copy_synchronize_cursor_end`](#window_copy_synchronize_cursor_end)


---
### window_copy_update_cursor<!-- {{#callable:window_copy_update_cursor}} -->
Updates the cursor position in the window copy mode.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure that contains the current window pane and mode data.
    - `cx`: The new x-coordinate (column) for the cursor.
    - `cy`: The new y-coordinate (row) for the cursor.
- **Control Flow**:
    - Store the current cursor position in `old_cx` and `old_cy`.
    - Update the cursor position in the `data` structure with the new coordinates `cx` and `cy`.
    - If the old cursor position was at the right edge of the screen, redraw the line at `old_cy`.
    - If the new cursor position is at the right edge of the screen, redraw the line at the new `cy`.
    - If the new cursor position is not at the right edge, start a screen write context and move the cursor to the new position.
- **Output**:
    - The function does not return a value; it modifies the state of the window copy mode by updating the cursor position and potentially redrawing lines.
- **Functions called**:
    - [`window_copy_redraw_lines`](#window_copy_redraw_lines)


---
### window_copy_start_selection<!-- {{#callable:window_copy_start_selection}} -->
The `window_copy_start_selection` function initiates a selection in the window copy mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the state of the window copy mode.
- **Control Flow**:
    - The function retrieves the `window_copy_mode_data` structure from the `window_mode_entry`.
    - It sets the starting coordinates of the selection (`selx`, `sely`) to the current cursor position adjusted for the screen offset.
    - The ending coordinates of the selection (`endselx`, `endsely`) are initialized to the same values as the starting coordinates.
    - The cursor drag mode is set to `CURSORDRAG_ENDSEL` to indicate that the end of the selection will follow the cursor.
    - Finally, it calls [`window_copy_set_selection`](#window_copy_set_selection) to update the selection state.
- **Output**:
    - The function does not return a value but modifies the selection state in the `window_copy_mode_data` structure.
- **Functions called**:
    - [`window_copy_set_selection`](#window_copy_set_selection)


---
### window_copy_adjust_selection<!-- {{#callable:window_copy_adjust_selection}} -->
Adjusts the selection coordinates in copy mode based on the current cursor position.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `selx`: A pointer to an unsigned integer representing the x-coordinate of the selection.
    - `sely`: A pointer to an unsigned integer representing the y-coordinate of the selection.
- **Control Flow**:
    - Retrieve the current selection coordinates from `selx` and `sely`.
    - Calculate the top edge of the visible screen area using `screen_hsize` and `data->oy`.
    - Check if the y-coordinate of the selection is above the visible area, and if so, set the relative position to 'above' and adjust the selection coordinates accordingly.
    - Check if the y-coordinate of the selection is below the visible area, and if so, set the relative position to 'below' and adjust the selection coordinates accordingly.
    - If the selection is within the visible area, set the relative position to 'on screen' and adjust the y-coordinate based on the top edge.
- **Output**:
    - Returns an integer indicating the relative position of the selection: above, on screen, or below.


---
### window_copy_update_selection<!-- {{#callable:window_copy_update_selection}} -->
Updates the selection in window copy mode.
- **Inputs**:
    - `wme`: Pointer to the `window_mode_entry` structure representing the current window mode.
    - `may_redraw`: An integer flag indicating whether the screen may be redrawn.
    - `no_reset`: An integer flag indicating whether to reset the selection.
- **Control Flow**:
    - Checks if there is no selection and the line flag is set to LINE_SEL_NONE; if true, returns 0.
    - Calls the [`window_copy_set_selection`](#window_copy_set_selection) function to update the selection based on the provided parameters.
- **Output**:
    - Returns an integer indicating the success or failure of the selection update.
- **Functions called**:
    - [`window_copy_set_selection`](#window_copy_set_selection)


---
### window_copy_set_selection<!-- {{#callable:window_copy_set_selection}} -->
The `window_copy_set_selection` function adjusts and sets the selection in a window copy mode, handling visual updates and selection styles.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `may_redraw`: An integer flag indicating whether the screen may be redrawn after setting the selection.
    - `no_reset`: An integer flag indicating whether to reset the cursor position.
- **Control Flow**:
    - The function begins by synchronizing the cursor position with the current selection state.
    - It adjusts the start and end positions of the selection based on the current cursor position.
    - If the selection is outside the visible screen, it hides the selection and returns 0.
    - It applies the selection style and sets the selection on the screen.
    - If the selection is rectangular and redrawing is allowed, it calculates which lines need to be redrawn based on the selection's position.
- **Output**:
    - The function returns 1 if the selection was successfully set, or 0 if the selection was hidden.
- **Functions called**:
    - [`window_copy_synchronize_cursor`](#window_copy_synchronize_cursor)
    - [`window_copy_adjust_selection`](#window_copy_adjust_selection)
    - [`window_copy_redraw_lines`](#window_copy_redraw_lines)


---
### window_copy_get_selection<!-- {{#callable:window_copy_get_selection}} -->
The `window_copy_get_selection` function retrieves the currently selected text from a window in copy mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window pane and mode data.
    - `len`: A pointer to a size_t variable where the length of the selected text will be stored.
- **Control Flow**:
    - Checks if there is a selection; if not, it attempts to match text at the cursor position.
    - If a selection exists, it determines the start and end coordinates of the selection.
    - Handles rectangular selections differently based on the cursor's position.
    - Copies the selected lines into a buffer, adjusting for the selection's start and end points.
    - Trims the output if necessary and returns the buffer containing the selected text.
- **Output**:
    - Returns a pointer to a dynamically allocated buffer containing the selected text, or NULL if no selection exists, while also updating the length of the selection in the provided `len` pointer.
- **Functions called**:
    - [`window_copy_match_at_cursor`](#window_copy_match_at_cursor)
    - [`window_copy_find_length`](#window_copy_find_length)
    - [`window_copy_copy_line`](#window_copy_copy_line)


---
### window_copy_copy_buffer<!-- {{#callable:window_copy_copy_buffer}} -->
Copies a buffer to the clipboard or paste buffer based on specified flags.
- **Inputs**:
    - `wme`: Pointer to the `window_mode_entry` structure representing the current window mode.
    - `prefix`: A string prefix to be added to the copied content.
    - `buf`: Pointer to the buffer containing the data to be copied.
    - `len`: Size of the buffer to be copied.
    - `set_paste`: Integer flag indicating whether to set the paste buffer.
    - `set_clip`: Integer flag indicating whether to set the clipboard.
- **Control Flow**:
    - Checks if `set_clip` is true and if the global clipboard option is enabled.
    - If so, starts a screen write context for the pane, sets the selection with the buffer, and notifies the pane.
    - If `set_paste` is true, adds the buffer to the paste buffer with the specified prefix.
- **Output**:
    - The function does not return a value; it performs actions based on the input parameters.


---
### window_copy_pipe_run<!-- {{#callable:window_copy_pipe_run}} -->
The `window_copy_pipe_run` function executes a command to process the current selection from a window copy mode.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `s`: A pointer to the `session` structure representing the current session.
    - `cmd`: A string containing the command to be executed.
    - `len`: A pointer to a size_t variable that will hold the length of the selection.
- **Control Flow**:
    - Calls [`window_copy_get_selection`](#window_copy_get_selection) to retrieve the current selection and its length.
    - If `cmd` is NULL or an empty string, it retrieves the default copy command from global options.
    - If a valid command is provided, it runs the command as a job using `job_run`.
    - Writes the selection buffer to the job's event using `bufferevent_write`.
    - Returns the buffer containing the selection.
- **Output**:
    - Returns a pointer to the buffer containing the copied selection.
- **Functions called**:
    - [`window_copy_get_selection`](#window_copy_get_selection)


---
### window_copy_pipe<!-- {{#callable:window_copy_pipe}} -->
The `window_copy_pipe` function executes a command to pipe the contents of the current selection in the window copy mode.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `s`: A pointer to the `session` structure representing the current session.
    - `cmd`: A string containing the command to be executed for piping the selection.
- **Control Flow**:
    - Calls [`window_copy_pipe_run`](#window_copy_pipe_run) to execute the command with the current selection.
    - The length of the selection is passed by reference to [`window_copy_pipe_run`](#window_copy_pipe_run).
- **Output**:
    - The function does not return a value, but it triggers the execution of a command that processes the selected text.
- **Functions called**:
    - [`window_copy_pipe_run`](#window_copy_pipe_run)


---
### window_copy_copy_pipe<!-- {{#callable:window_copy_copy_pipe}} -->
The `window_copy_copy_pipe` function copies data from a window's copy mode to a specified command pipeline.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `s`: A pointer to the `session` structure representing the current session.
    - `prefix`: A string prefix to be added to the copied data.
    - `cmd`: A command string that specifies the pipeline to which the data will be sent.
    - `set_paste`: An integer flag indicating whether to set the clipboard for pasting.
    - `set_clip`: An integer flag indicating whether to set the clipboard.
- **Control Flow**:
    - The function calls [`window_copy_pipe_run`](#window_copy_pipe_run) to execute the command and retrieve the data to be copied.
    - If the returned buffer is not NULL, it calls [`window_copy_copy_buffer`](#window_copy_copy_buffer) to copy the data to the specified command pipeline.
- **Output**:
    - The function does not return a value; it performs actions based on the input parameters to copy data to a command pipeline.
- **Functions called**:
    - [`window_copy_pipe_run`](#window_copy_pipe_run)
    - [`window_copy_copy_buffer`](#window_copy_copy_buffer)


---
### window_copy_copy_selection<!-- {{#callable:window_copy_copy_selection}} -->
Copies the currently selected text in a window to a buffer.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `prefix`: A string prefix to be added to the copied content.
    - `set_paste`: An integer flag indicating whether to set the clipboard for pasting.
    - `set_clip`: An integer flag indicating whether to set the clipboard.
- **Control Flow**:
    - Calls [`window_copy_get_selection`](#window_copy_get_selection) to retrieve the selected text and its length.
    - If the retrieved buffer is not NULL, it calls [`window_copy_copy_buffer`](#window_copy_copy_buffer) to copy the selection to the clipboard.
- **Output**:
    - The function does not return a value; it performs actions to copy the selected text to the clipboard.
- **Functions called**:
    - [`window_copy_get_selection`](#window_copy_get_selection)
    - [`window_copy_copy_buffer`](#window_copy_copy_buffer)


---
### window_copy_append_selection<!-- {{#callable:window_copy_append_selection}} -->
The `window_copy_append_selection` function appends the current selection from a window to the paste buffer.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window pane and mode data.
- **Control Flow**:
    - Calls [`window_copy_get_selection`](#window_copy_get_selection) to retrieve the current selection and its length.
    - If the selection buffer is not empty, it checks if clipboard integration is enabled and sets the clipboard selection.
    - Retrieves the top paste buffer and appends the current selection to it if it exists.
    - If the paste buffer is not empty, it reallocates memory to accommodate the new data and combines it with the existing data.
    - Finally, it attempts to set the updated paste buffer, freeing the buffer if the operation fails.
- **Output**:
    - The function does not return a value; it modifies the paste buffer directly.
- **Functions called**:
    - [`window_copy_get_selection`](#window_copy_get_selection)


---
### window_copy_copy_line<!-- {{#callable:window_copy_copy_line}} -->
The `window_copy_copy_line` function copies a specified range of characters from a line in a grid to a buffer.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure that contains the state of the window copy mode.
    - `buf`: A pointer to a character buffer where the copied line will be stored.
    - `off`: A pointer to a size_t variable that tracks the current offset in the buffer.
    - `sy`: The y-coordinate of the line in the grid to be copied.
    - `sx`: The starting x-coordinate from which to begin copying.
    - `ex`: The ending x-coordinate up to which to copy characters.
- **Control Flow**:
    - The function first checks if the starting x-coordinate (sx) is greater than the ending x-coordinate (ex) and returns if true.
    - It retrieves the line from the grid corresponding to the y-coordinate (sy) and checks if the line is wrapped.
    - If the line is wrapped and fully visible, it sets the length to the full cell size; otherwise, it calculates the visible length of the line.
    - It adjusts the ending x-coordinate (ex) if it exceeds the calculated length and similarly adjusts the starting x-coordinate (sx).
    - If the starting x-coordinate is less than the ending x-coordinate, it enters a loop to copy each character from the grid to the buffer.
    - During the copy, it checks for padding and tab characters, handling them appropriately.
    - Finally, if the line was not wrapped, it appends a newline character to the buffer.
- **Output**:
    - The function does not return a value but modifies the buffer pointed to by `buf` to contain the copied characters and updates the offset pointed to by `off`.
- **Functions called**:
    - [`window_copy_find_length`](#window_copy_find_length)


---
### window_copy_clear_selection<!-- {{#callable:window_copy_clear_selection}} -->
The `window_copy_clear_selection` function clears the current selection in the window copy mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the state of the window copy mode.
- **Control Flow**:
    - Calls `screen_clear_selection` to clear the selection on the screen associated with the `window_copy_mode_data`.
    - Resets the `cursordrag` state to `CURSORDRAG_NONE`.
    - Sets the `lineflag` to `LINE_SEL_NONE` and `selflag` to `SEL_CHAR`.
    - Calculates the current cursor position in the backing screen and updates the cursor position if it exceeds the line length.
- **Output**:
    - The function does not return a value; it modifies the state of the selection in the window copy mode.
- **Functions called**:
    - [`window_copy_find_length`](#window_copy_find_length)
    - [`window_copy_update_cursor`](#window_copy_update_cursor)


---
### window_copy_in_set<!-- {{#callable:window_copy_in_set}} -->
The `window_copy_in_set` function checks if a specific grid cell at given coordinates is part of a specified character set.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
    - `px`: An unsigned integer representing the x-coordinate of the grid cell.
    - `py`: An unsigned integer representing the y-coordinate of the grid cell.
    - `set`: A pointer to a character string representing the set of characters to check against.
- **Control Flow**:
    - The function retrieves the `window_copy_mode_data` structure from the `window_mode_entry` structure.
    - It then calls the `grid_in_set` function, passing the grid associated with the backing data, the x and y coordinates, and the character set to check.
    - The result of the `grid_in_set` function is returned as the output.
- **Output**:
    - Returns an integer indicating whether the specified grid cell is part of the given character set (non-zero if true, zero otherwise).


---
### window_copy_find_length<!-- {{#callable:window_copy_find_length}} -->
The `window_copy_find_length` function retrieves the length of a specific line in a grid associated with a window copy mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
    - `py`: An unsigned integer representing the line number in the grid for which the length is to be determined.
- **Control Flow**:
    - The function accesses the `data` field of the `wme` structure to get the associated `window_copy_mode_data`.
    - It then calls the `grid_line_length` function, passing the grid from the `backing` field of `data` and the line number `py` to retrieve the length of the specified line.
- **Output**:
    - Returns an unsigned integer representing the length of the specified line in the grid.


---
### window_copy_cursor_start_of_line<!-- {{#callable:window_copy_cursor_start_of_line}} -->
The `window_copy_cursor_start_of_line` function moves the cursor to the start of the current line in a window copy mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
- **Control Flow**:
    - Retrieve the `window_copy_mode_data` structure from the `window_mode_entry`.
    - Calculate the current cursor position in the backing screen.
    - Initialize a `grid_reader` to read the grid data from the backing screen.
    - Use the `grid_reader` to move the cursor to the start of the current line.
    - Get the new cursor position after moving to the start of the line.
    - Call [`window_copy_acquire_cursor_up`](#window_copy_acquire_cursor_up) to update the cursor position in the window copy mode.
- **Output**:
    - The function does not return a value; it modifies the cursor position in the window copy mode.
- **Functions called**:
    - [`window_copy_acquire_cursor_up`](#window_copy_acquire_cursor_up)


---
### window_copy_cursor_back_to_indentation<!-- {{#callable:window_copy_cursor_back_to_indentation}} -->
The `window_copy_cursor_back_to_indentation` function moves the cursor in a window copy mode back to the indentation level of the current line.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
- **Control Flow**:
    - The function retrieves the current cursor position and the backing screen data.
    - It initializes a `grid_reader` to read the grid of the backing screen.
    - The cursor is moved back to the indentation level using the `grid_reader_cursor_back_to_indentation` function.
    - The new cursor position is then retrieved and the cursor is updated accordingly.
- **Output**:
    - The function does not return a value; it updates the cursor position in the window copy mode based on the indentation level of the current line.
- **Functions called**:
    - [`window_copy_acquire_cursor_up`](#window_copy_acquire_cursor_up)


---
### window_copy_cursor_end_of_line<!-- {{#callable:window_copy_cursor_end_of_line}} -->
The `window_copy_cursor_end_of_line` function moves the cursor to the end of the current line in a window copy mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the state of the window copy mode.
- **Control Flow**:
    - Retrieve the `window_copy_mode_data` structure from the `window_mode_entry` to access the current cursor position and backing screen.
    - Calculate the current cursor position in the backing screen based on the current cursor coordinates.
    - Initialize a `grid_reader` to read the grid of the backing screen starting from the current cursor position.
    - Check if there is a selection and if rectangle selection mode is active; if so, move the cursor to the end of the line with selection, otherwise move it without selection.
    - Get the new cursor position after moving to the end of the line.
    - Call [`window_copy_acquire_cursor_down`](#window_copy_acquire_cursor_down) to update the cursor position in the window copy mode.
- **Output**:
    - The function does not return a value but updates the cursor position in the window copy mode based on the end of the current line.
- **Functions called**:
    - [`window_copy_acquire_cursor_down`](#window_copy_acquire_cursor_down)


---
### window_copy_other_end<!-- {{#callable:window_copy_other_end}} -->
The `window_copy_other_end` function toggles the selection end point in a window copy mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the state of the window copy mode.
- **Control Flow**:
    - Checks if there is a selection; if not, the function returns immediately.
    - Toggles the selection line direction between left-right and right-left.
    - Updates the cursor drag state based on the current selection state.
    - Determines the current selection end coordinates based on the cursor drag state.
    - Calculates the new cursor position based on the selection end coordinates and the screen size.
    - Updates the selection and redraws the screen to reflect the changes.
- **Output**:
    - The function does not return a value but modifies the state of the window copy mode and updates the display accordingly.
- **Functions called**:
    - [`window_copy_update_selection`](#window_copy_update_selection)
    - [`window_copy_redraw_screen`](#window_copy_redraw_screen)


---
### window_copy_cursor_left<!-- {{#callable:window_copy_cursor_left}} -->
Moves the cursor one position to the left in the window copy mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the state of the window copy mode.
- **Control Flow**:
    - Retrieve the current cursor position (cx, cy) and the height of the screen (hsize).
    - Calculate the current position in the backing grid based on the cursor's y-coordinate.
    - Use a grid reader to move the cursor left by one position.
    - Update the cursor's position in the window copy mode based on the new coordinates.
- **Output**:
    - The function does not return a value; it modifies the cursor's position in the window copy mode.
- **Functions called**:
    - [`window_copy_acquire_cursor_up`](#window_copy_acquire_cursor_up)


---
### window_copy_cursor_right<!-- {{#callable:window_copy_cursor_right}} -->
The `window_copy_cursor_right` function moves the cursor one position to the right in the window copy mode.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `all`: An integer flag indicating whether to move the cursor across all characters, including any special characters.
- **Control Flow**:
    - The function retrieves the current cursor position from the `window_copy_mode_data` structure.
    - It calculates the effective position in the backing screen based on the current cursor position.
    - A `grid_reader` is initialized to read the grid of the backing screen.
    - The cursor is moved to the right by calling `grid_reader_cursor_right` with the specified parameters.
    - The new cursor position is retrieved from the grid reader.
    - Finally, the cursor is updated in the window copy mode using [`window_copy_acquire_cursor_down`](#window_copy_acquire_cursor_down).
- **Output**:
    - The function does not return a value; it updates the cursor position in the window copy mode based on the movement.
- **Functions called**:
    - [`window_copy_acquire_cursor_down`](#window_copy_acquire_cursor_down)


---
### window_copy_cursor_up<!-- {{#callable:window_copy_cursor_up}} -->
The `window_copy_cursor_up` function moves the cursor up in the window copy mode, handling scrolling and selection updates.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `scroll_only`: An integer flag indicating whether to only scroll without moving the cursor.
- **Control Flow**:
    - Check if there is no rectangular selection or if the selection is not active.
    - Calculate the current cursor position and the length of the line at that position.
    - If there is no selection and the cursor is not at the end of the line, update the last cursor position.
    - If the cursor is at the top of the screen or if scrolling is requested, scroll down the screen.
    - If not scrolling, update the cursor position based on the current selection state.
    - If the cursor is not in a rectangular selection, adjust the cursor position based on the line length.
    - If the line selection mode is active, update the cursor position accordingly.
- **Output**:
    - The function does not return a value but updates the cursor position and potentially redraws the screen based on the new cursor location.
- **Functions called**:
    - [`window_copy_find_length`](#window_copy_find_length)
    - [`window_copy_other_end`](#window_copy_other_end)
    - [`window_copy_scroll_down`](#window_copy_scroll_down)
    - [`window_copy_redraw_lines`](#window_copy_redraw_lines)
    - [`window_copy_update_cursor`](#window_copy_update_cursor)
    - [`window_copy_update_selection`](#window_copy_update_selection)


---
### window_copy_cursor_down<!-- {{#callable:window_copy_cursor_down}} -->
The `window_copy_cursor_down` function moves the cursor down in a copy mode interface, handling scrolling and selection updates.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
    - `scroll_only`: An integer flag indicating whether to only scroll without moving the cursor.
- **Control Flow**:
    - Retrieve the current cursor position and selection state from the `window_copy_mode_data` structure.
    - Calculate the current line's offset and length to determine if the cursor is within a selection.
    - If `scroll_only` is true or the cursor is at the bottom of the screen, scroll the view up by one line.
    - If not scrolling, update the cursor position based on the current selection state.
    - Check if the cursor position exceeds the line length and adjust accordingly.
    - Handle different line selection modes to update the cursor position and selection state.
- **Output**:
    - The function does not return a value but updates the cursor position and the selection state in the window copy mode.
- **Functions called**:
    - [`window_copy_find_length`](#window_copy_find_length)
    - [`window_copy_other_end`](#window_copy_other_end)
    - [`window_copy_scroll_up`](#window_copy_scroll_up)
    - [`window_copy_redraw_lines`](#window_copy_redraw_lines)
    - [`window_copy_update_cursor`](#window_copy_update_cursor)
    - [`window_copy_update_selection`](#window_copy_update_selection)


---
### window_copy_cursor_jump<!-- {{#callable:window_copy_cursor_jump}} -->
The `window_copy_cursor_jump` function moves the cursor in the window copy mode to the next occurrence of a specified jump character.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
- **Control Flow**:
    - The function retrieves the current cursor position and the jump character from the `window_copy_mode_data` structure.
    - It initializes a `grid_reader` to read the grid of the backing screen starting from the cursor's current position.
    - The function attempts to jump to the next occurrence of the specified jump character using `grid_reader_cursor_jump`.
    - If a jump is successful, it retrieves the new cursor position and updates the cursor's position in the window copy mode.
- **Output**:
    - The function does not return a value; it directly modifies the cursor position in the window copy mode.
- **Functions called**:
    - [`window_copy_acquire_cursor_down`](#window_copy_acquire_cursor_down)


---
### window_copy_cursor_jump_back<!-- {{#callable:window_copy_cursor_jump_back}} -->
The `window_copy_cursor_jump_back` function moves the cursor back to the previous occurrence of a specified jump character in the window copy mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current state of the window copy mode.
- **Control Flow**:
    - The function retrieves the current cursor position and the backing screen from the `window_mode_entry` structure.
    - It initializes a `grid_reader` to read the grid of the backing screen starting from the current cursor position.
    - The cursor is moved one position to the left to prepare for the jump back operation.
    - If a previous occurrence of the jump character is found, the cursor position is updated to that location.
    - Finally, the cursor is moved up to the new position in the window copy mode.
- **Output**:
    - The function does not return a value; it modifies the cursor position in the window copy mode based on the jump character.
- **Functions called**:
    - [`window_copy_acquire_cursor_up`](#window_copy_acquire_cursor_up)


---
### window_copy_cursor_jump_to<!-- {{#callable:window_copy_cursor_jump_to}} -->
The `window_copy_cursor_jump_to` function moves the cursor to the next occurrence of a specified jump character in the window copy mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
- **Control Flow**:
    - The function retrieves the current cursor position and the backing screen data.
    - It initializes a `grid_reader` to read the grid data starting from the cursor's current position.
    - It attempts to jump to the next occurrence of the specified jump character using `grid_reader_cursor_jump`.
    - If a jump is successful, it retrieves the new cursor position and updates the cursor's position in the window copy mode.
- **Output**:
    - The function does not return a value; it modifies the cursor position in the window copy mode based on the jump character.
- **Functions called**:
    - [`window_copy_acquire_cursor_down`](#window_copy_acquire_cursor_down)


---
### window_copy_cursor_jump_to_back<!-- {{#callable:window_copy_cursor_jump_to_back}} -->
The `window_copy_cursor_jump_to_back` function moves the cursor to the previous occurrence of a specified jump character in the backing screen.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
- **Control Flow**:
    - The function retrieves the current cursor position and the backing screen's height.
    - It initializes a `grid_reader` to read the backing grid starting from the current cursor position.
    - The cursor is moved two positions to the left to prepare for the jump search.
    - If a previous occurrence of the jump character is found, the cursor is moved to that position.
    - The new cursor position is then updated in the window copy mode data.
- **Output**:
    - The function does not return a value; it updates the cursor position in the window copy mode data based on the jump character search.
- **Functions called**:
    - [`window_copy_acquire_cursor_up`](#window_copy_acquire_cursor_up)


---
### window_copy_cursor_next_word<!-- {{#callable:window_copy_cursor_next_word}} -->
The `window_copy_cursor_next_word` function moves the cursor to the next word in a text buffer, defined by specified separators.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
    - `separators`: A string containing characters that are considered as word separators.
- **Control Flow**:
    - The function retrieves the current cursor position and the height of the screen.
    - It initializes a `grid_reader` to read the text grid from the current cursor position.
    - The cursor is moved to the next word based on the provided separators using the `grid_reader_cursor_next_word` function.
    - The new cursor position is then retrieved from the grid reader.
    - Finally, the cursor is updated in the window mode entry using the [`window_copy_acquire_cursor_down`](#window_copy_acquire_cursor_down) function.
- **Output**:
    - The function does not return a value; it updates the cursor position in the window mode entry.
- **Functions called**:
    - [`window_copy_acquire_cursor_down`](#window_copy_acquire_cursor_down)


---
### window_copy_cursor_next_word_end_pos<!-- {{#callable:window_copy_cursor_next_word_end_pos}} -->
The `window_copy_cursor_next_word_end_pos` function calculates the cursor's position at the end of the next word based on the current cursor position and specified word separators.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure that contains the current window pane and mode data.
    - `separators`: A string containing characters that are considered as word separators.
    - `ppx`: A pointer to an unsigned integer where the x-coordinate of the cursor's new position will be stored.
    - `ppy`: A pointer to an unsigned integer where the y-coordinate of the cursor's new position will be stored.
- **Control Flow**:
    - The function retrieves the current cursor position from the `data` structure.
    - It initializes a `grid_reader` to read the grid of the backing screen.
    - If the mode keys are set to 'vi', it checks if the cursor is currently in a whitespace character and moves the cursor right if it is not.
    - The function then calls `grid_reader_cursor_next_word_end` to move the cursor to the end of the next word based on the provided separators.
    - After determining the new cursor position, it retrieves the updated cursor coordinates and stores them in the provided pointers.
- **Output**:
    - The function does not return a value; instead, it updates the x and y coordinates of the cursor's position through the provided pointers.


---
### window_copy_cursor_next_word_end<!-- {{#callable:window_copy_cursor_next_word_end}} -->
The `window_copy_cursor_next_word_end` function moves the cursor to the end of the next word in a copy mode interface.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
    - `separators`: A string containing characters that are considered as word separators.
    - `no_reset`: An integer flag indicating whether to reset the cursor position.
- **Control Flow**:
    - Retrieve the current cursor position and the height of the screen.
    - Initialize a `grid_reader` to read the grid of the backing screen.
    - Check if the mode keys are set to 'vi'. If so, adjust the cursor position if not on whitespace.
    - Move the cursor to the end of the next word using the provided separators.
    - Adjust the cursor position back by one character to position it correctly at the end of the word.
    - Acquire the new cursor position and update the window mode entry with the new cursor coordinates.
- **Output**:
    - The function does not return a value but updates the cursor position in the window mode entry.
- **Functions called**:
    - [`window_copy_acquire_cursor_down`](#window_copy_acquire_cursor_down)


---
### window_copy_cursor_previous_word_pos<!-- {{#callable:window_copy_cursor_previous_word_pos}} -->
The `window_copy_cursor_previous_word_pos` function updates the cursor position to the start of the previous word in a text grid.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
    - `separators`: A string containing characters that are considered as word separators.
    - `ppx`: A pointer to an unsigned integer where the new cursor x position will be stored.
    - `ppy`: A pointer to an unsigned integer where the new cursor y position will be stored.
- **Control Flow**:
    - Retrieve the current cursor position from the `data` structure.
    - Calculate the effective y-coordinate by adjusting for the screen offset.
    - Initialize a grid reader to start reading from the current cursor position.
    - Use the grid reader to move the cursor to the previous word based on the provided separators.
    - Get the new cursor position from the grid reader and store it in the provided pointers.
- **Output**:
    - The function does not return a value but updates the x and y cursor positions through the provided pointers.


---
### window_copy_cursor_previous_word<!-- {{#callable:window_copy_cursor_previous_word}} -->
The `window_copy_cursor_previous_word` function moves the cursor to the beginning of the previous word in a copy mode interface.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
    - `separators`: A string containing characters that are considered as word separators.
    - `already`: An integer indicating whether the cursor should stop at the end of the line.
- **Control Flow**:
    - The function retrieves the current cursor position and the height of the screen.
    - It checks the mode of the key bindings to determine if it should stop at the end of the line.
    - A grid reader is initialized to read the grid data from the current screen.
    - The cursor is moved to the previous word based on the provided separators and the already flag.
    - The new cursor position is acquired from the grid reader.
    - Finally, the cursor is updated to the new position in the window copy mode.
- **Output**:
    - The function does not return a value; it updates the cursor position in the window copy mode.
- **Functions called**:
    - [`window_copy_acquire_cursor_up`](#window_copy_acquire_cursor_up)


---
### window_copy_cursor_prompt<!-- {{#callable:window_copy_cursor_prompt}} -->
The `window_copy_cursor_prompt` function updates the cursor position in a copy mode window based on the specified direction.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
    - `direction`: An integer indicating the direction to move the cursor: 0 for up and 1 for down.
    - `start_output`: An integer that determines the type of line to start output from, indicating whether to start from a prompt or output line.
- **Control Flow**:
    - The function retrieves the current cursor position and the backing grid from the `window_mode_entry` structure.
    - It determines the line flag based on the `start_output` parameter, which indicates whether to start from a prompt or output line.
    - Depending on the `direction` parameter, it sets the increment value for moving the cursor and defines the end line for the movement.
    - If the current line is equal to the end line, the function returns immediately without making any changes.
    - A loop is initiated to move the cursor line by line in the specified direction until a line with the appropriate flag is found or the end line is reached.
    - The cursor's x and y coordinates are updated based on the new line position, and the selection is updated accordingly.
    - Finally, the screen is redrawn to reflect the updated cursor position.
- **Output**:
    - The function does not return a value but updates the cursor position and redraws the screen to reflect the changes.
- **Functions called**:
    - [`window_copy_update_selection`](#window_copy_update_selection)
    - [`window_copy_redraw_screen`](#window_copy_redraw_screen)


---
### window_copy_scroll_up<!-- {{#callable:window_copy_scroll_up}} -->
The `window_copy_scroll_up` function scrolls the content of a window copy mode up by a specified number of lines.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `ny`: An unsigned integer representing the number of lines to scroll up.
- **Control Flow**:
    - Check if the current offset `data->oy` is less than `ny`, and if so, set `ny` to `data->oy`.
    - If `ny` is zero, return immediately as there is nothing to scroll.
    - Decrement `data->oy` by `ny` to adjust the offset.
    - If there are search marks and no timeout, call [`window_copy_search_marks`](#window_copy_search_marks) to update the search marks.
    - Call [`window_copy_update_selection`](#window_copy_update_selection) to refresh the selection state.
    - Start a screen write context for the window pane.
    - Move the cursor to the top left corner of the screen.
    - Delete `ny` lines from the top of the screen.
    - Write the lines from the window copy mode to the screen, adjusting for the new offset.
    - Move the cursor back to its original position.
    - Stop the screen write context.
    - Set the redraw scrollbar flag for the window pane.
- **Output**:
    - The function does not return a value but updates the window's display by scrolling the content up and potentially modifying the selection and search marks.
- **Functions called**:
    - [`window_copy_search_marks`](#window_copy_search_marks)
    - [`window_copy_update_selection`](#window_copy_update_selection)
    - [`window_copy_write_lines`](#window_copy_write_lines)
    - [`window_copy_write_line`](#window_copy_write_line)


---
### window_copy_scroll_down<!-- {{#callable:window_copy_scroll_down}} -->
The `window_copy_scroll_down` function scrolls the content of a window copy mode down by a specified number of lines.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `ny`: An unsigned integer representing the number of lines to scroll down.
- **Control Flow**:
    - Check if `ny` exceeds the height of the backing screen; if so, return immediately.
    - Adjust `ny` if the current offset plus `ny` exceeds the height of the backing screen.
    - If `ny` is zero after adjustment, return immediately.
    - Increment the offset `oy` by `ny` to reflect the scroll down.
    - If there are search marks and no timeout, call [`window_copy_search_marks`](#window_copy_search_marks) to update the search marks.
    - Call [`window_copy_update_selection`](#window_copy_update_selection) to refresh the selection state.
    - Start a screen write context for the window pane.
    - Move the cursor to the top left corner of the screen.
    - Insert `ny` lines into the screen to create space for the scroll.
    - Write the lines from the window copy mode to the screen context.
    - If there is a selection, write the appropriate line to the screen.
    - Move the cursor back to its original position and stop the screen write context.
    - Set the redraw scrollbar flag for the window pane.
- **Output**:
    - The function does not return a value; it modifies the state of the window copy mode by scrolling the content down and updating the display accordingly.
- **Functions called**:
    - [`window_copy_search_marks`](#window_copy_search_marks)
    - [`window_copy_update_selection`](#window_copy_update_selection)
    - [`window_copy_write_lines`](#window_copy_write_lines)
    - [`window_copy_write_line`](#window_copy_write_line)


---
### window_copy_rectangle_set<!-- {{#callable:window_copy_rectangle_set}} -->
Sets the rectangle copy mode in the window copy mode.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure that contains the state of the window copy mode.
    - `rectflag`: An integer flag indicating whether rectangle copy mode should be enabled (1) or disabled (0).
- **Control Flow**:
    - Sets the `rectflag` in the `window_copy_mode_data` structure to the value of `rectflag` input.
    - Calculates the current cursor position in the backing screen based on the current cursor coordinates and the offset.
    - Determines the maximum length of the current line at the cursor's y-coordinate.
    - If the current cursor x-coordinate exceeds the maximum line length, updates the cursor position to the maximum length.
    - Updates the selection state and redraws the screen to reflect the changes.
- **Output**:
    - The function does not return a value; it modifies the state of the window copy mode and updates the display accordingly.
- **Functions called**:
    - [`window_copy_find_length`](#window_copy_find_length)
    - [`window_copy_update_cursor`](#window_copy_update_cursor)
    - [`window_copy_update_selection`](#window_copy_update_selection)
    - [`window_copy_redraw_screen`](#window_copy_redraw_screen)


---
### window_copy_move_mouse<!-- {{#callable:window_copy_move_mouse}} -->
The `window_copy_move_mouse` function updates the cursor position in a window copy mode based on mouse events.
- **Inputs**:
    - `m`: A pointer to a `mouse_event` structure that contains information about the mouse event.
- **Control Flow**:
    - The function first retrieves the window pane associated with the mouse event using `cmd_mouse_pane`.
    - If no valid window pane is found, the function returns immediately.
    - It then checks if there are any active modes in the window pane; if none are found, it returns.
    - The function verifies that the current mode is either `window_copy_mode` or `window_view_mode`.
    - Next, it checks if the mouse event's coordinates correspond to a valid position in the window pane using `cmd_mouse_at`.
    - If the coordinates are valid, it updates the cursor position using [`window_copy_update_cursor`](#window_copy_update_cursor).
- **Output**:
    - The function does not return a value; it modifies the state of the window copy mode by updating the cursor position based on the mouse event.
- **Functions called**:
    - [`window_copy_update_cursor`](#window_copy_update_cursor)


---
### window_copy_start_drag<!-- {{#callable:window_copy_start_drag}} -->
The `window_copy_start_drag` function initiates a drag selection in a window copy mode based on mouse events.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client that initiated the drag.
    - `m`: A pointer to the `mouse_event` structure containing information about the mouse event.
- **Control Flow**:
    - The function first checks if the `client` pointer `c` is NULL and returns if it is.
    - It retrieves the `window_pane` associated with the mouse event using `cmd_mouse_pane`.
    - If the `window_pane` is NULL, it returns immediately.
    - It checks if the first mode in the `window_pane` is either `window_copy_mode` or `window_view_mode`, returning if it is not.
    - The function then checks the mouse position and updates the cursor position based on the mouse coordinates.
    - It sets the drag update and release functions for the client.
    - The selection type is determined based on the mouse position relative to the current selection.
    - Depending on the selection type (character, word, or line), it updates the cursor position accordingly.
    - Finally, it redraws the screen and calls the drag update function.
- **Output**:
    - The function does not return a value but updates the state of the window copy mode, including the cursor position and selection state.
- **Functions called**:
    - [`window_copy_update_cursor`](#window_copy_update_cursor)
    - [`window_copy_cursor_previous_word_pos`](#window_copy_cursor_previous_word_pos)
    - [`window_copy_start_selection`](#window_copy_start_selection)
    - [`window_copy_redraw_screen`](#window_copy_redraw_screen)
    - [`window_copy_drag_update`](#window_copy_drag_update)


---
### window_copy_drag_update<!-- {{#callable:window_copy_drag_update}} -->
The `window_copy_drag_update` function updates the cursor position and selection in a window copy mode based on mouse drag events.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client that initiated the drag event.
    - `m`: A pointer to the `mouse_event` structure containing information about the mouse event.
- **Control Flow**:
    - The function first checks if the `client` pointer is NULL and returns if it is.
    - It retrieves the `window_pane` associated with the mouse event using `cmd_mouse_pane`.
    - If the `window_pane` is NULL, it returns immediately.
    - It checks if the current mode is either `window_copy_mode` or `window_view_mode` and returns if not.
    - The function updates the cursor position based on the mouse coordinates.
    - It updates the selection if the cursor position has changed.
    - If the cursor is at the top or bottom of the screen, it sets a timer to move the cursor up or down after a delay.
- **Output**:
    - The function does not return a value but updates the cursor position and selection visually in the window copy mode.
- **Functions called**:
    - [`window_copy_update_cursor`](#window_copy_update_cursor)
    - [`window_copy_update_selection`](#window_copy_update_selection)
    - [`window_copy_redraw_selection`](#window_copy_redraw_selection)
    - [`window_copy_cursor_up`](#window_copy_cursor_up)
    - [`window_copy_cursor_down`](#window_copy_cursor_down)


---
### window_copy_drag_release<!-- {{#callable:window_copy_drag_release}} -->
The `window_copy_drag_release` function handles the release of a mouse drag event in the window copy mode.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client that initiated the drag.
    - `m`: A pointer to the `mouse_event` structure containing details about the mouse event.
- **Control Flow**:
    - The function first checks if the `client` pointer `c` is NULL, returning immediately if it is.
    - It retrieves the associated `window_pane` using the `cmd_mouse_pane` function with the mouse event.
    - If the `window_pane` is NULL, the function returns.
    - It then retrieves the first `window_mode_entry` from the `window_pane`'s modes.
    - If there is no mode entry or if the mode is not `window_copy_mode` or `window_view_mode`, it returns.
    - Finally, it cancels any active drag timer by calling `evtimer_del` on the `dragtimer` associated with the `window_copy_mode_data`.
- **Output**:
    - The function does not return a value; it performs actions based on the state of the mouse drag and cleans up any associated timers.


---
### window_copy_jump_to_mark<!-- {{#callable:window_copy_jump_to_mark}} -->
The `window_copy_jump_to_mark` function updates the cursor position in a window copy mode to a previously marked location.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data.
- **Control Flow**:
    - The function retrieves the current cursor position and the mark position from the `window_copy_mode_data` structure.
    - It updates the cursor's x-coordinate (`cx`) to the mark's x-coordinate (`mx`).
    - It checks if the mark's y-coordinate (`my`) is within the visible screen height; if it is, it sets the cursor's y-coordinate (`cy`) to 0 and adjusts the offset (`oy`) accordingly.
    - If the mark's y-coordinate is below the visible screen height, it sets the cursor's y-coordinate (`cy`) based on the mark's position and resets the offset (`oy`) to 0.
    - Finally, it updates the mark's position to the previous cursor position and marks it as visible, then calls functions to update the selection and redraw the screen.
- **Output**:
    - The function does not return a value; it modifies the state of the `window_copy_mode_data` structure and updates the display accordingly.
- **Functions called**:
    - [`window_copy_update_selection`](#window_copy_update_selection)
    - [`window_copy_redraw_screen`](#window_copy_redraw_screen)


---
### window_copy_acquire_cursor_up<!-- {{#callable:window_copy_acquire_cursor_up}} -->
The `window_copy_acquire_cursor_up` function adjusts the cursor position in a window copy mode by moving it up and updating the selection and display accordingly.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `hsize`: An unsigned integer representing the height of the window.
    - `oy`: An unsigned integer representing the number of lines scrolled up.
    - `oldy`: An unsigned integer representing the previous cursor y-coordinate.
    - `px`: An unsigned integer representing the x-coordinate of the cursor.
    - `py`: An unsigned integer representing the y-coordinate of the cursor.
- **Control Flow**:
    - Calculate the current cursor position relative to the height of the window and the number of lines scrolled up.
    - Determine how many lines to move the cursor up based on the current position and the height of the window.
    - Move the cursor up by calling [`window_copy_cursor_up`](#window_copy_cursor_up) in a loop until the desired position is reached.
    - Update the cursor position using [`window_copy_update_cursor`](#window_copy_update_cursor) with the new coordinates.
    - If the selection is updated, redraw the affected lines using [`window_copy_redraw_lines`](#window_copy_redraw_lines).
- **Output**:
    - The function does not return a value but updates the cursor position and potentially redraws the window content.
- **Functions called**:
    - [`window_copy_cursor_up`](#window_copy_cursor_up)
    - [`window_copy_update_cursor`](#window_copy_update_cursor)
    - [`window_copy_update_selection`](#window_copy_update_selection)
    - [`window_copy_redraw_lines`](#window_copy_redraw_lines)


---
### window_copy_acquire_cursor_down<!-- {{#callable:window_copy_acquire_cursor_down}} -->
The `window_copy_acquire_cursor_down` function adjusts the cursor position downwards in a window copy mode, handling scrolling and selection updates.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode.
    - `hsize`: An unsigned integer representing the height size of the window.
    - `sy`: An unsigned integer representing the total screen height.
    - `oy`: An unsigned integer representing the offset of the cursor.
    - `oldy`: An unsigned integer representing the previous cursor y-coordinate.
    - `px`: An unsigned integer representing the x-coordinate of the cursor.
    - `py`: An unsigned integer representing the y-coordinate of the cursor.
    - `no_reset`: An integer flag indicating whether to reset the selection.
- **Control Flow**:
    - Calculate the current cursor position based on the provided coordinates and height size.
    - Determine if the cursor is beyond the visible screen height and calculate the necessary adjustments.
    - If the cursor is beyond the screen height, scroll the cursor down by the required number of lines.
    - Update the cursor position based on the new calculated coordinates.
    - Update the selection if necessary, and redraw the affected lines on the screen.
- **Output**:
    - The function does not return a value but updates the cursor position and potentially redraws the screen.
- **Functions called**:
    - [`window_copy_cursor_down`](#window_copy_cursor_down)
    - [`window_copy_update_cursor`](#window_copy_update_cursor)
    - [`window_copy_update_selection`](#window_copy_update_selection)
    - [`window_copy_redraw_lines`](#window_copy_redraw_lines)


# Function Declarations (Public API)

---
### window_copy_key_table<!-- {{#callable_declaration:window_copy_key_table}} -->
Returns the key table for the window copy mode.
- **Description**: This function retrieves the appropriate key table string based on the current mode keys setting of the associated window pane. It should be called when initializing or configuring the window copy mode to ensure the correct key bindings are used. The function checks the 'mode-keys' option of the window's options and returns 'copy-mode-vi' if the mode is set to VI, otherwise it returns 'copy-mode'. This function does not modify any input parameters.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. Must not be null.
- **Output**: Returns a pointer to a string representing the key table for the current copy mode, which can be either 'copy-mode-vi' or 'copy-mode'.
- **See also**: [`window_copy_key_table`](#window_copy_key_table)  (Implementation)


---
### window_copy_command<!-- {{#callable_declaration:window_copy_command}} -->
Processes a command in window copy mode.
- **Description**: This function is called to execute a command in the window copy mode, which allows users to interact with the text displayed in a terminal window. It should be invoked when a command is entered, and it handles various actions based on the command provided. The function expects that the window mode entry is already initialized and that the command is valid. If the command is not recognized or if there are issues with the arguments, appropriate actions are taken, such as clearing selections or redrawing the screen. The function also manages mouse events if applicable.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode. Must not be null.
    - `c`: A pointer to the `client` structure representing the client that initiated the command. Must not be null.
    - `s`: A pointer to the `session` structure representing the current session. Must not be null.
    - `wl`: A pointer to the `winlink` structure representing the current window link. Must not be null.
    - `args`: A pointer to the `args` structure containing the command arguments. Must not be null.
    - `m`: A pointer to the `mouse_event` structure representing the mouse event, if applicable. Can be null.
- **Output**: Returns nothing. The function may modify the state of the window copy mode, including updating the display or handling selections.
- **See also**: [`window_copy_command`](#window_copy_command)  (Implementation)


---
### window_copy_init<!-- {{#callable_declaration:window_copy_init}} -->
Initializes the window copy mode.
- **Description**: This function sets up the window copy mode for a specified window pane, allowing users to copy text from the pane. It must be called when entering copy mode, typically after the window pane has been initialized. The function handles various parameters, including scroll behavior and hyperlink management, and prepares the screen for displaying the copied content. It is important to ensure that the window pane is valid and that the function is called in the appropriate context to avoid undefined behavior.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the current window mode. Must not be null.
    - `fs`: A pointer to a `cmd_find_state` structure, which is unused in this function. Can be null.
    - `args`: A pointer to an `args` structure containing command-line arguments. The function expects this to be valid and properly initialized.
- **Output**: Returns a pointer to a `screen` structure that represents the initialized screen for the window copy mode.
- **See also**: [`window_copy_init`](#window_copy_init)  (Implementation)


---
### window_copy_view_init<!-- {{#callable_declaration:window_copy_view_init}} -->
Initializes a view mode for window copy functionality.
- **Description**: This function sets up the window copy mode in view mode, which allows users to view the contents of a window pane without modifying it. It must be called with a valid `window_mode_entry` structure that has been properly initialized. The function allocates necessary resources for managing the view mode, including initializing the backing screen and input context. It is important to ensure that the `window_mode_entry` is correctly set up before calling this function, as it relies on the state of the associated window pane.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure that represents the current window mode. This must not be null and should be properly initialized before calling the function.
    - `fs`: An unused pointer to a `cmd_find_state` structure. This parameter is not utilized in this function.
    - `args`: An unused pointer to an `args` structure. This parameter is not utilized in this function.
- **Output**: Returns a pointer to a `screen` structure that represents the initialized screen for the view mode. This screen can be used for rendering the contents of the window pane in view mode.
- **See also**: [`window_copy_view_init`](#window_copy_view_init)  (Implementation)


---
### window_copy_free<!-- {{#callable_declaration:window_copy_free}} -->
Frees resources associated with a window copy mode.
- **Description**: This function should be called to release all resources allocated for a `window_mode_entry` when it is no longer needed. It is essential to call this function to prevent memory leaks, especially after the window copy mode has been exited. The function handles the deallocation of various components, including search marks, strings, and screen buffers. It is important to ensure that the `window_mode_entry` passed to this function is valid and has been properly initialized before calling this function.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry`. This must not be null and should point to a valid window mode entry that has been initialized.
- **Output**: None
- **See also**: [`window_copy_free`](#window_copy_free)  (Implementation)


---
### window_copy_resize<!-- {{#callable_declaration:window_copy_resize}} -->
Resizes the window copy mode display.
- **Description**: This function should be called to adjust the size of the window copy mode display when the terminal window is resized. It updates the internal screen representation and cursor position based on the new dimensions. The function expects that the window copy mode is currently active and that the dimensions provided are valid. If the new size is smaller than the current size, the content may be reflowed to fit the new dimensions. The function also triggers a redraw of the screen to reflect the changes.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` representing the current window mode. Must not be null.
    - `sx`: An unsigned integer representing the new width of the window. Must be greater than zero.
    - `sy`: An unsigned integer representing the new height of the window. Must be greater than zero.
- **Output**: None
- **See also**: [`window_copy_resize`](#window_copy_resize)  (Implementation)


---
### window_copy_formats<!-- {{#callable_declaration:window_copy_formats}} -->
Copies format data from a window mode entry to a format tree.
- **Description**: This function is used to populate a `format_tree` with various attributes related to the current state of a window in copy mode. It should be called when you need to retrieve or display information about the copy mode, such as the cursor position, selection details, and search status. The function expects that the `window_mode_entry` has been properly initialized and that the `format_tree` is ready to receive format data. It handles various edge cases, such as the presence of selections and search results, and updates the format tree accordingly.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure that contains the current state of the window in copy mode. Must not be null.
    - `ft`: A pointer to a `format_tree` structure where the format data will be added. Must not be null.
- **Output**: None
- **See also**: [`window_copy_formats`](#window_copy_formats)  (Implementation)


---
### window_copy_get_screen<!-- {{#callable_declaration:window_copy_get_screen}} -->
Returns the screen associated with the window copy mode.
- **Description**: This function retrieves the `screen` structure that is used as the backing store for the window copy mode. It should be called when the window copy mode is active, and it provides access to the screen's content for further manipulation or display. Ensure that the `window_mode_entry` structure passed to this function is valid and properly initialized, as passing a null or uninitialized pointer may lead to undefined behavior.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure that represents the current state of the window copy mode. This pointer must not be null and should point to a valid entry that has been initialized.
- **Output**: Returns a pointer to the `screen` structure associated with the window copy mode. This pointer can be used to access the content of the screen.
- **See also**: [`window_copy_get_screen`](#window_copy_get_screen)  (Implementation)


---
### window_copy_scroll1<!-- {{#callable_declaration:window_copy_scroll1}} -->
Handles scrolling in the window copy mode.
- **Description**: This function is used to adjust the view of the content in the window copy mode based on the position of the scrollbar slider. It must be called during a drag event when the user interacts with the scrollbar. The function calculates the new position of the content based on the slider's position and updates the view accordingly. If the scrolling reaches the top or bottom of the content, it may trigger an exit from the copy mode.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode. Must not be null.
    - `wp`: A pointer to the `window_pane` structure representing the pane being manipulated. Must not be null.
    - `sl_mpos`: An integer representing the position of the mouse relative to the scrollbar slider. Valid values depend on the scrollbar's height.
    - `my`: An unsigned integer representing the current mouse Y position. Must be within the bounds of the window pane.
    - `scroll_exit`: An integer flag indicating whether to exit the copy mode when scrolling reaches the top. Can be 0 (do not exit) or 1 (exit on reaching the top).
- **Output**: None
- **See also**: [`window_copy_scroll1`](#window_copy_scroll1)  (Implementation)


---
### window_copy_pageup1<!-- {{#callable_declaration:window_copy_pageup1}} -->
Scrolls the window copy mode up by a specified amount.
- **Description**: This function is used to scroll the content displayed in the window copy mode. It can be called to scroll up by either a full page or half a page, depending on the value of the `half_page` parameter. The function adjusts the cursor position and updates the selection if necessary. It is important to ensure that this function is called when the window copy mode is active, and it should not be called if the content is already at the top of the scrollable area.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. Must not be null.
    - `half_page`: An integer that determines the amount to scroll. If non-zero, scrolls half a page; otherwise, scrolls a full page. Valid values are 0 or 1.
- **Output**: None
- **See also**: [`window_copy_pageup1`](#window_copy_pageup1)  (Implementation)


---
### window_copy_pagedown1<!-- {{#callable_declaration:window_copy_pagedown1}} -->
Copies the displayed content down by a specified amount.
- **Description**: This function is used to scroll the content of a window down by a specified number of lines or half a page. It should be called when the window is in copy mode, and it can be used to navigate through the content displayed in the window. The `half_page` parameter determines whether to scroll down by half the window height or the full height minus two lines. The `scroll_exit` parameter indicates whether to exit the copy mode when the scroll reaches the bottom of the content. If the scrolling reaches the end of the content and `scroll_exit` is set, the function will return 1, indicating that the copy mode should be exited.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the current window mode. Must not be null.
    - `half_page`: An integer indicating whether to scroll down by half a page (1) or a full page (0). Valid values are 0 or 1.
    - `scroll_exit`: An integer indicating whether to exit copy mode when the scroll reaches the bottom (1) or not (0). Valid values are 0 or 1.
- **Output**: Returns 1 if the scroll reaches the bottom and `scroll_exit` is set, otherwise returns 0.
- **See also**: [`window_copy_pagedown1`](#window_copy_pagedown1)  (Implementation)


---
### window_copy_next_paragraph<!-- {{#callable_declaration:window_copy_next_paragraph}} -->
Scrolls to the next paragraph in the window.
- **Description**: This function is used to navigate through text in a window by scrolling to the next paragraph. It should be called when the user wants to move the view downwards in the text. The function will skip over any empty lines until it finds the next non-empty line, effectively moving the cursor to the start of the next paragraph. It is important to ensure that the window is properly initialized before calling this function.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry`. This structure must be valid and properly initialized before calling the function.
- **Output**: The function does not return a value and does not modify any inputs directly.
- **See also**: [`window_copy_next_paragraph`](#window_copy_next_paragraph)  (Implementation)


---
### window_copy_previous_paragraph<!-- {{#callable_declaration:window_copy_previous_paragraph}} -->
Scrolls to the previous paragraph in the window.
- **Description**: This function is used to navigate to the previous paragraph in a window copy mode. It should be called when the user wants to move the cursor up to the start of the previous paragraph. The function will adjust the cursor position based on the current line and the content of the window. It is important to ensure that the window is in copy mode before calling this function, as it relies on the structure of the window's content to determine paragraph boundaries.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. This parameter must not be null and should point to a valid window mode entry that is in copy mode.
- **Output**: None
- **See also**: [`window_copy_previous_paragraph`](#window_copy_previous_paragraph)  (Implementation)


---
### window_copy_redraw_selection<!-- {{#callable_declaration:window_copy_redraw_selection}} -->
Redraws the selection in the window copy mode.
- **Description**: This function is used to update the visual representation of the selection area in the window copy mode. It should be called whenever the selection changes, specifically when the cursor moves or the selection is adjusted. The function takes into account the previous cursor position to determine which lines need to be redrawn. It also handles special cases for word selection mode, where the first word on the line below the cursor may also be selected. It is important to ensure that this function is called in a context where the window copy mode is active.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode. Must not be null.
    - `old_y`: An unsigned integer representing the previous y-coordinate of the cursor. This value is used to determine the range of lines that need to be redrawn.
- **Output**: None
- **See also**: [`window_copy_redraw_selection`](#window_copy_redraw_selection)  (Implementation)


---
### window_copy_redraw_lines<!-- {{#callable_declaration:window_copy_redraw_lines}} -->
Redraws a specified range of lines in a window.
- **Description**: This function is used to refresh the display of a specific range of lines in a window copy mode. It should be called when the content of the window has changed and needs to be updated visually. The function takes the starting line and the number of lines to redraw as parameters. It is important to ensure that the window is in a valid state before calling this function, as it assumes that the window mode entry is properly initialized and that the specified lines are within the bounds of the window's content.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode. Must not be null.
    - `py`: The starting line number to redraw. Must be within the valid range of lines in the window.
    - `ny`: The number of lines to redraw, starting from `py`. Must be a positive integer.
- **Output**: None
- **See also**: [`window_copy_redraw_lines`](#window_copy_redraw_lines)  (Implementation)


---
### window_copy_redraw_screen<!-- {{#callable_declaration:window_copy_redraw_screen}} -->
Redraws the entire screen for the window copy mode.
- **Description**: This function is intended to be called when the screen needs to be refreshed in the window copy mode. It redraws all lines from the top of the screen to the bottom, ensuring that the current state of the copy mode is accurately reflected. It should be called after any changes that affect the display, such as resizing the window or updating the content. There are no specific preconditions, but it is important to ensure that the window copy mode is active when this function is invoked.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. This pointer must not be null.
- **Output**: None
- **See also**: [`window_copy_redraw_screen`](#window_copy_redraw_screen)  (Implementation)


---
### window_copy_write_line<!-- {{#callable_declaration:window_copy_write_line}} -->
Writes a line of text to the window copy mode.
- **Description**: This function is used to render a specific line of text in the window copy mode, which is typically invoked during the display of copied content. It should be called within the context of a window mode entry that has been properly initialized. The function handles the application of styles to the text based on the current options set for the window pane. It also manages the display of a position indicator if configured to do so, and it ensures that the cursor is correctly positioned after the line is written.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode. Must not be null.
    - `ctx`: A pointer to the `screen_write_ctx` structure used for writing to the screen. Must not be null.
    - `py`: An unsigned integer representing the line number to be written. Valid range is from 0 to the height of the screen.
- **Output**: None
- **See also**: [`window_copy_write_line`](#window_copy_write_line)  (Implementation)


---
### window_copy_write_lines<!-- {{#callable_declaration:window_copy_write_lines}} -->
Writes multiple lines to the screen.
- **Description**: This function is used to write a specified number of lines, starting from a given line index, to the screen in a window copy mode. It is typically called when the screen needs to be updated with new content, such as after scrolling or resizing. The function assumes that the window mode entry and screen write context are properly initialized and that the specified line range is valid. If the specified range exceeds the available lines, it will only write up to the maximum available lines.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode. Must not be null.
    - `ctx`: A pointer to the `screen_write_ctx` structure used for writing to the screen. Must not be null.
    - `py`: An unsigned integer representing the starting line index from which to begin writing. Must be within the bounds of the screen.
    - `ny`: An unsigned integer representing the number of lines to write. Must be greater than zero.
- **Output**: None
- **See also**: [`window_copy_write_lines`](#window_copy_write_lines)  (Implementation)


---
### window_copy_match_at_cursor<!-- {{#callable_declaration:window_copy_match_at_cursor}} -->
Copies the matched text at the cursor position.
- **Description**: This function is used to retrieve and copy the text that matches the current search mark at the cursor's position in the window copy mode. It should be called when a search has been performed and a match is expected at the cursor's location. The function will return NULL if there is no search mark or if the cursor is not positioned over a valid match. The caller is responsible for managing the memory of the returned string, which will contain the matched text, including handling any necessary reallocations.
- **Inputs**:
    - `data`: A pointer to a `struct window_copy_mode_data` that contains the state of the window copy mode. Must not be null.
- **Output**: Returns a pointer to a dynamically allocated string containing the matched text. The caller must free this string when it is no longer needed.
- **See also**: [`window_copy_match_at_cursor`](#window_copy_match_at_cursor)  (Implementation)


---
### window_copy_scroll_to<!-- {{#callable_declaration:window_copy_scroll_to}} -->
Scrolls the view to the specified position.
- **Description**: This function is used to adjust the scroll position of the window copy mode to a specific coordinate defined by the parameters. It should be called when the user wants to navigate to a specific point in the text. The parameters `px` and `py` define the new cursor position, while the `no_redraw` flag indicates whether the screen should be redrawn after the scroll operation. If `no_redraw` is set to 0, the screen will be updated to reflect the new cursor position, which is useful for providing immediate visual feedback to the user.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode. Must not be null.
    - `px`: An unsigned integer representing the new x-coordinate for the cursor position. Valid values depend on the width of the window.
    - `py`: An unsigned integer representing the new y-coordinate for the cursor position. Valid values depend on the height of the window.
    - `no_redraw`: An integer flag that determines whether the screen should be redrawn after scrolling. A value of 0 indicates that the screen should be updated, while a non-zero value indicates that it should not.
- **Output**: The function does not return a value. It modifies the state of the window copy mode by updating the cursor position and potentially redrawing the screen.
- **See also**: [`window_copy_scroll_to`](#window_copy_scroll_to)  (Implementation)


---
### window_copy_search_compare<!-- {{#callable_declaration:window_copy_search_compare}} -->
Compares two grid cells for equality.
- **Description**: This function is used to compare the contents of two grid cells located at specified coordinates in two different grids. It is particularly useful in scenarios where you need to determine if the contents of one cell match another, such as during search operations in a text-based interface. The comparison can be case-insensitive if specified. The function should be called with valid grid pointers and coordinates, and it will return a non-zero value if the cells are considered equal, and zero otherwise.
- **Inputs**:
    - `gd`: A pointer to the first `grid` structure. Must not be null.
    - `px`: The x-coordinate of the cell in the first grid. Must be within the bounds of the grid.
    - `py`: The y-coordinate of the cell in the first grid. Must be within the bounds of the grid.
    - `sgd`: A pointer to the second `grid` structure. Must not be null.
    - `spx`: The x-coordinate of the cell in the second grid. Must be within the bounds of the grid.
    - `cis`: An integer flag indicating whether the comparison should be case-insensitive (non-zero) or case-sensitive (zero).
- **Output**: Returns a non-zero value if the cells are equal, and zero if they are not.
- **See also**: [`window_copy_search_compare`](#window_copy_search_compare)  (Implementation)


---
### window_copy_search_lr<!-- {{#callable_declaration:window_copy_search_lr}} -->
Searches for a pattern in a grid.
- **Description**: This function searches for a specified pattern in a source grid (`sgd`) within a specified range of the target grid (`gd`). It starts searching from the specified `first` index to the `last` index along a specified row (`py`). The search is case-insensitive if the `cis` parameter is set. If a match is found, the function updates the output parameter `ppx` with the index of the match and returns 1. If no match is found, it returns 0. The function should be called with valid grid pointers and indices, and it is expected that the grids are properly initialized.
- **Inputs**:
    - `gd`: Pointer to the target grid where the search will be performed. Must not be null.
    - `sgd`: Pointer to the source grid containing the pattern to search for. Must not be null.
    - `ppx`: Pointer to an unsigned integer where the index of the match will be stored. Caller retains ownership.
    - `py`: The row index in the target grid to start searching from. Valid range is from 0 to the height of the grid.
    - `first`: The starting index for the search in the target grid. Must be less than or equal to 'last'.
    - `last`: The ending index for the search in the target grid. Must be greater than 'first'.
    - `cis`: An integer indicating whether the search should be case-insensitive (non-zero) or case-sensitive (zero).
- **Output**: Returns 1 if a match is found and updates `ppx` with the index of the match; returns 0 if no match is found.
- **See also**: [`window_copy_search_lr`](#window_copy_search_lr)  (Implementation)


---
### window_copy_search_rl<!-- {{#callable_declaration:window_copy_search_rl}} -->
Searches for a substring in a grid.
- **Description**: This function searches for a substring within a specified range of a grid, comparing it against another grid. It is typically used in scenarios where text needs to be located within a grid structure, such as in a text editor or terminal multiplexer. The search is performed in a right-to-left direction, and the function will return a success indicator if the substring is found. The parameters must be valid, with the `gd` and `sgd` grids properly initialized, and the `ppx` pointer must not be null. If the search range is invalid or if the substring is not found, the function will return 0.
- **Inputs**:
    - `gd`: A pointer to the grid structure where the search will be performed. Must not be null.
    - `sgd`: A pointer to the grid structure containing the substring to search for. Must not be null.
    - `ppx`: A pointer to an unsigned integer where the position of the found substring will be stored. Must not be null.
    - `py`: The y-coordinate in the grid where the search will start. Must be within the bounds of the grid.
    - `first`: The starting index of the search range. Must be less than or equal to 'last'.
    - `last`: The ending index of the search range. Must be greater than 'first'.
    - `cis`: An integer indicating case sensitivity. Non-zero values indicate case-insensitive search.
- **Output**: Returns 1 if the substring is found, and 0 if it is not found or if the search parameters are invalid.
- **See also**: [`window_copy_search_rl`](#window_copy_search_rl)  (Implementation)


---
### window_copy_last_regex<!-- {{#callable_declaration:window_copy_last_regex}} -->
Copies the last match of a regular expression from a grid.
- **Description**: This function is used to search for the last occurrence of a specified regular expression within a given buffer and copy the matching content's position to the provided output parameters. It should be called with a valid `grid` structure and a compiled regular expression. The function will return 1 if a match is found and the positions are updated; otherwise, it returns 0. The search is constrained by the specified range defined by the `first` and `last` parameters, and it respects the line boundaries of the grid. If the match is found, the output parameters `ppx` and `psx` will contain the position of the match and its length, respectively.
- **Inputs**:
    - `gd`: A pointer to a `grid` structure representing the text grid to search. Must not be null.
    - `py`: The y-coordinate in the grid from which to start searching. Must be within the grid's height.
    - `first`: The starting x-coordinate for the search range. Must be less than or equal to `last`.
    - `last`: The ending x-coordinate for the search range. Must be greater than or equal to `first`.
    - `len`: The length of the buffer to search within. Must be greater than zero.
    - `ppx`: A pointer to a `u_int` where the x-coordinate of the match will be stored. Caller retains ownership.
    - `psx`: A pointer to a `u_int` where the length of the match will be stored. Caller retains ownership.
    - `buf`: A pointer to a character buffer containing the text to search. Must not be null.
    - `preg`: A pointer to a compiled `regex_t` structure representing the regular expression to match. Must not be null.
    - `eflags`: Flags for the regex execution, which can modify the behavior of the search. Valid values depend on the regex implementation.
- **Output**: Returns 1 if a match is found and updates `ppx` and `psx` with the match position and length, respectively. Returns 0 if no match is found.
- **See also**: [`window_copy_last_regex`](#window_copy_last_regex)  (Implementation)


---
### window_copy_search_mark_at<!-- {{#callable_declaration:window_copy_search_mark_at}} -->
Calculates the position in a grid based on pixel coordinates.
- **Description**: This function is used to determine the position in a grid based on the provided pixel coordinates (`px`, `py`). It should be called when the pixel coordinates are known and the caller needs to find the corresponding position in the grid. The function checks if the provided `py` coordinate is within the valid range of the grid; if it is not, it returns -1. If valid, it calculates the corresponding position and stores it in the `at` parameter. The function assumes that the `data` structure has been properly initialized and that the grid is set up correctly.
- **Inputs**:
    - `data`: A pointer to a `window_copy_mode_data` structure that contains the grid information. Must not be null.
    - `px`: The pixel x-coordinate. Valid values are any non-negative integer.
    - `py`: The pixel y-coordinate. Must be within the range of the grid's height. If out of bounds, the function returns -1.
    - `at`: A pointer to a `u_int` where the calculated position will be stored. The caller retains ownership.
- **Output**: Returns 0 on success, indicating that the position was successfully calculated and stored in `at`. Returns -1 if the `py` coordinate is out of the valid range.
- **See also**: [`window_copy_search_mark_at`](#window_copy_search_mark_at)  (Implementation)


---
### window_copy_stringify<!-- {{#callable_declaration:window_copy_stringify}} -->
Copies a specified range of characters from a grid to a buffer.
- **Description**: This function is used to copy a substring from a specified line in a grid, defined by the parameters `first` and `last`, into a provided buffer. The function dynamically resizes the buffer if the required size exceeds the current buffer size. It is important to ensure that the `buf` parameter is initially allocated and that `size` points to a valid size variable. The function will append a null terminator to the end of the copied string. If the specified range is invalid, the function will handle it gracefully by not performing any copy operations.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure from which the string will be copied. Must not be null.
    - `py`: The line number in the grid from which to start copying. Must be a valid line index.
    - `first`: The starting index of the characters to copy from the specified line. Must be less than or equal to `last`.
    - `last`: The ending index of the characters to copy from the specified line. Must be greater than `first`.
    - `buf`: A pointer to the buffer where the copied string will be stored. Must be allocated by the caller.
    - `size`: A pointer to a variable that holds the size of the buffer. The function will update this value to reflect the new size after copying.
- **Output**: Returns a pointer to the updated buffer containing the copied string.
- **See also**: [`window_copy_stringify`](#window_copy_stringify)  (Implementation)


---
### window_copy_cstrtocellpos<!-- {{#callable_declaration:window_copy_cstrtocellpos}} -->
Maps a C string to a cell position in a grid.
- **Description**: This function is used to find the cell position in a grid that corresponds to a given C string. It is particularly useful in scenarios where you need to locate text within a grid structure, such as in a text editor or terminal emulator. The function takes a grid, the number of cells to consider, and pointers to the current cell position (x and y coordinates) as well as the string to search for. It is important to ensure that the grid is properly initialized before calling this function. If the string is not found, the function will set the position to one past the end of the grid.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid in which to search. Must not be null.
    - `ncells`: The number of cells to consider in the search. Must be greater than zero.
    - `ppx`: A pointer to an unsigned integer representing the x-coordinate of the cell position. The function will update this value to the found position.
    - `ppy`: A pointer to an unsigned integer representing the y-coordinate of the cell position. The function will update this value to the found position.
    - `str`: A pointer to a null-terminated C string to search for within the grid. Must not be null.
- **Output**: The function does not return a value, but it updates the values pointed to by `ppx` and `ppy` to reflect the found cell position corresponding to the input string.
- **See also**: [`window_copy_cstrtocellpos`](#window_copy_cstrtocellpos)  (Implementation)


---
### window_copy_search_marks<!-- {{#callable_declaration:window_copy_search_marks}} -->
Searches for marks in the window copy mode.
- **Description**: This function is used to search for specific marks within the content of a window in copy mode. It can be called with a specified screen to search within, and it supports both regular expression and plain text searches. The function handles visible-only searches and manages timeouts for long-running searches. It is important to ensure that the search string is set before calling this function, and it should be noted that the function may modify the state of the search marks. If the search is successful, it updates the search count and may indicate if more matches are available.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode. Must not be null.
    - `ssp`: A pointer to the `screen` structure to search within. If null, a temporary screen is created for the search.
    - `regex`: An integer flag indicating whether to treat the search string as a regular expression (1) or plain text (0). Valid values are 0 or 1.
    - `visible_only`: An integer flag indicating whether to search only within visible lines (1) or all lines (0). Valid values are 0 or 1.
- **Output**: Returns 1 if the search was successful, 0 if no matches were found or if an error occurred.
- **See also**: [`window_copy_search_marks`](#window_copy_search_marks)  (Implementation)


---
### window_copy_clear_marks<!-- {{#callable_declaration:window_copy_clear_marks}} -->
Clears all search marks in the window copy mode.
- **Description**: This function is intended to be called when you want to remove any existing search marks from the window copy mode. It should be used when the user no longer needs the search marks, such as after a search operation is completed or when the user exits the copy mode. The function does not return any value and does not modify any parameters.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. This parameter must not be null.
- **Output**: None
- **See also**: [`window_copy_clear_marks`](#window_copy_clear_marks)  (Implementation)


---
### window_copy_is_lowercase<!-- {{#callable_declaration:window_copy_is_lowercase}} -->
Checks if all characters in the string are lowercase.
- **Description**: This function is used to determine if a given string consists entirely of lowercase alphabetic characters. It iterates through each character in the string until it encounters a null terminator. If any character is found that is not lowercase, the function returns 0. If all characters are lowercase, it returns 1. The input string must not be null.
- **Inputs**:
    - `ptr`: A pointer to a null-terminated string. Must not be null. The function will return 0 if the string is null.
- **Output**: Returns 1 if all characters in the string are lowercase; otherwise, returns 0.
- **See also**: [`window_copy_is_lowercase`](#window_copy_is_lowercase)  (Implementation)


---
### window_copy_search_back_overlap<!-- {{#callable_declaration:window_copy_search_back_overlap}} -->
Handles backward wrapped regex searches with overlapping matches.
- **Description**: This function is used to search for a regex pattern in a grid, specifically when the search is performed backward and may involve wrapped lines. It is particularly useful in scenarios where the search pattern may overlap with previous matches in wrapped lines. The function modifies the provided pointers to update the position of the found match, if any, and ensures that the search respects the boundaries of the grid. It should be called with valid parameters, and the grid must be properly initialized.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the text grid to search. Must not be null.
    - `preg`: A pointer to a compiled regex pattern (`regex_t`). Must not be null.
    - `ppx`: A pointer to an unsigned integer that will be updated to the x-coordinate of the found match. Caller retains ownership.
    - `psx`: A pointer to an unsigned integer that will be updated to the size of the match found. Caller retains ownership.
    - `ppy`: A pointer to an unsigned integer that will be updated to the y-coordinate of the found match. Caller retains ownership.
    - `endline`: An unsigned integer representing the last line to search. Must be a valid line index within the grid.
- **Output**: The function does not return a value. It modifies the values pointed to by `ppx`, `psx`, and `ppy` to reflect the position and size of the found match.
- **See also**: [`window_copy_search_back_overlap`](#window_copy_search_back_overlap)  (Implementation)


---
### window_copy_search_jump<!-- {{#callable_declaration:window_copy_search_jump}} -->
Searches for text in a specified direction.
- **Description**: This function is used to search for a specified text within a grid, starting from a given position and moving in a specified direction. It can handle both regular expressions and plain text searches, with options for case sensitivity and wrapping around the grid. The function should be called when the user wants to find occurrences of text, and it will update the cursor position to the found text if a match is found. If the search reaches the end of the grid and wrapping is enabled, it will continue searching from the beginning.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the current window mode. Must not be null.
    - `gd`: A pointer to a `grid` structure representing the grid to search within. Must not be null.
    - `sgd`: A pointer to a `grid` structure representing the source grid to search against. Must not be null.
    - `fx`: The x-coordinate (column) to start the search from. Must be within the bounds of the grid.
    - `fy`: The y-coordinate (row) to start the search from. Must be within the bounds of the grid.
    - `endline`: The last line to search up to. Must be within the bounds of the grid.
    - `cis`: An integer indicating case sensitivity; non-zero for case insensitive search.
    - `wrap`: An integer indicating whether to wrap around the grid when the end is reached; non-zero for wrapping.
    - `direction`: An integer indicating the search direction; non-zero for forward search, zero for backward search.
    - `regex`: An integer indicating whether to treat the search string as a regular expression; non-zero for regex.
- **Output**: Returns 1 if a match is found and the cursor is updated to the match position; returns 0 if no match is found.
- **See also**: [`window_copy_search_jump`](#window_copy_search_jump)  (Implementation)


---
### window_copy_search<!-- {{#callable_declaration:window_copy_search}} -->
Searches for a string in the window copy mode.
- **Description**: This function is used to search for a specified string within the window copy mode. It can search in either direction (up or down) and can utilize regular expressions if specified. The function should be called when the user wants to find occurrences of a string in the displayed text. Preconditions include that the search string must be set prior to calling this function. If the search string is empty or invalid, the function will return without performing a search. The function also handles wrapping around the text if the search reaches the end or beginning.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode. Must not be null.
    - `direction`: An integer indicating the search direction: 0 for up and 1 for down. Valid values are 0 or 1.
    - `regex`: An integer indicating whether to treat the search string as a regular expression (1) or not (0). Valid values are 0 or 1.
- **Output**: Returns 1 if the search string is found, 0 if not found, or if the search is invalid.
- **See also**: [`window_copy_search`](#window_copy_search)  (Implementation)


---
### window_copy_search_up<!-- {{#callable_declaration:window_copy_search_up}} -->
Searches upwards in the window copy mode.
- **Description**: This function is used to perform a search operation in the upward direction within the window copy mode. It should be called when the user wants to find a specific string or pattern in the text displayed in the window. The search is case-sensitive if the regex parameter is set to 1, and it will return to the previous match if called multiple times. The function expects that the window copy mode is already active, and it will not handle any initialization or state management for the window mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. Must not be null.
    - `regex`: An integer indicating whether the search should be treated as a regular expression (1) or a plain string (0). Valid values are 0 or 1.
- **Output**: Returns an integer indicating the success of the search operation. A non-zero value indicates that a match was found, while zero indicates no match.
- **See also**: [`window_copy_search_up`](#window_copy_search_up)  (Implementation)


---
### window_copy_search_down<!-- {{#callable_declaration:window_copy_search_down}} -->
Searches for a specified pattern in a window copy mode.
- **Description**: This function is used to search downwards for a specified pattern in the current window copy mode. It should be called when the user wants to find the next occurrence of a string or regular expression in the text being viewed. The function takes into account the current cursor position and searches from there downwards. If the search reaches the end of the text, it may wrap around to the beginning if the wrap option is enabled. The function does not modify any input parameters.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the current window mode. Must not be null.
    - `regex`: An integer indicating whether the search pattern is a regular expression (1) or a plain string (0). Valid values are 0 or 1.
- **Output**: Returns an integer indicating the success of the search operation. A non-zero value indicates that a match was found, while zero indicates no match.
- **See also**: [`window_copy_search_down`](#window_copy_search_down)  (Implementation)


---
### window_copy_goto_line<!-- {{#callable_declaration:window_copy_goto_line}} -->
Sets the cursor position to a specified line number.
- **Description**: This function is used to move the cursor to a specific line in the window copy mode. It takes a string representation of the line number as input, which is converted to an integer. The function should be called when the user wants to navigate to a particular line, and it ensures that the line number is within valid bounds. If the provided line number is invalid (negative or exceeds the total number of lines), it will adjust the cursor position to the nearest valid line. This function also updates the selection and redraws the screen to reflect the new cursor position.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the current window mode. Must not be null.
    - `linestr`: A string representing the line number to which the cursor should move. It must be a valid integer string; otherwise, the function will not perform any action.
- **Output**: None
- **See also**: [`window_copy_goto_line`](#window_copy_goto_line)  (Implementation)


---
### window_copy_update_cursor<!-- {{#callable_declaration:window_copy_update_cursor}} -->
Updates the cursor position in copy mode.
- **Description**: This function is used to change the cursor's position in the copy mode of a window. It should be called when the cursor needs to be moved to a new position specified by the parameters. The function handles edge cases such as moving the cursor to the end of the screen and ensures that the display is updated accordingly. It is important to note that this function should be called only when the window is in copy mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` representing the current window mode. Must not be null.
    - `cx`: An unsigned integer representing the new x-coordinate of the cursor. Valid values are from 0 to the width of the screen.
    - `cy`: An unsigned integer representing the new y-coordinate of the cursor. Valid values are from 0 to the height of the screen.
- **Output**: None
- **See also**: [`window_copy_update_cursor`](#window_copy_update_cursor)  (Implementation)


---
### window_copy_start_selection<!-- {{#callable_declaration:window_copy_start_selection}} -->
Starts a selection in copy mode.
- **Description**: This function is used to initiate a selection in copy mode, capturing the current cursor position as the starting point of the selection. It should be called when the user intends to select text for copying. The function updates the selection coordinates and sets the cursor drag state to indicate that the selection is in progress. It is important to ensure that this function is called in the appropriate context where a selection is expected.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. This parameter must not be null.
- **Output**: None
- **See also**: [`window_copy_start_selection`](#window_copy_start_selection)  (Implementation)


---
### window_copy_adjust_selection<!-- {{#callable_declaration:window_copy_adjust_selection}} -->
Adjusts the selection coordinates in the window copy mode.
- **Description**: This function is used to adjust the selection coordinates based on the current cursor position within the window copy mode. It should be called when the selection needs to be updated, such as during cursor movement. The function takes the current selection coordinates and modifies them based on their relation to the visible screen area. If the selection goes above or below the visible area, it will be adjusted accordingly. The function returns a value indicating the relative position of the selection with respect to the screen.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode. Must not be null.
    - `selx`: A pointer to an unsigned integer representing the x-coordinate of the selection. The caller retains ownership and must ensure it is valid.
    - `sely`: A pointer to an unsigned integer representing the y-coordinate of the selection. The caller retains ownership and must ensure it is valid.
- **Output**: Returns an integer indicating the relative position of the selection: above the visible area, on the screen, or below the visible area.
- **See also**: [`window_copy_adjust_selection`](#window_copy_adjust_selection)  (Implementation)


---
### window_copy_set_selection<!-- {{#callable_declaration:window_copy_set_selection}} -->
Sets the selection in copy mode.
- **Description**: This function is used to define the selection area in copy mode, adjusting the start and end points based on the current cursor position. It should be called after the cursor has been synchronized, and it handles cases where the selection may extend beyond the visible screen. If the selection is outside the current screen, it will hide the selection. The function also applies styles to the selection based on the configured options.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode. Must not be null.
    - `may_redraw`: An integer flag indicating whether the screen may be redrawn after setting the selection. Valid values are 0 (no redraw) and 1 (allow redraw).
    - `no_reset`: An integer flag indicating whether to reset the selection state. Valid values are 0 (reset) and 1 (do not reset).
- **Output**: Returns 1 if the selection was successfully set, or 0 if the selection is outside the current screen and thus hidden.
- **See also**: [`window_copy_set_selection`](#window_copy_set_selection)  (Implementation)


---
### window_copy_update_selection<!-- {{#callable_declaration:window_copy_update_selection}} -->
Updates the selection in the window copy mode.
- **Description**: This function should be called to refresh the selection state in the window copy mode. It checks if there is an active selection and updates it accordingly. If there is no selection and the line selection mode is not active, the function returns early. It is important to call this function after any changes to the selection or cursor position to ensure the selection is accurately represented.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode. Must not be null.
    - `may_redraw`: An integer flag indicating whether the screen may be redrawn. Valid values are 0 (no redraw) and 1 (allow redraw).
    - `no_reset`: An integer flag indicating whether to reset the selection state. Valid values are 0 (reset) and 1 (do not reset).
- **Output**: Returns an integer indicating the success of the operation. A return value of 0 indicates no selection was updated, while a non-zero value indicates that the selection was successfully updated.
- **See also**: [`window_copy_update_selection`](#window_copy_update_selection)  (Implementation)


---
### window_copy_synchronize_cursor<!-- {{#callable_declaration:window_copy_synchronize_cursor}} -->
Synchronizes the cursor position in copy mode.
- **Description**: This function is used to synchronize the cursor position based on the current selection mode. It should be called when the cursor is moved, and it adjusts the selection endpoints accordingly. The function handles different cursor drag states, ensuring that the selection is updated correctly based on whether the selection is being started, adjusted, or ended. It is important to note that this function should be called in the context of a window mode entry that has been properly initialized.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the current window mode. Must not be null.
    - `no_reset`: An integer flag indicating whether to reset the selection. Valid values are 0 (reset) or 1 (do not reset).
- **Output**: None
- **See also**: [`window_copy_synchronize_cursor`](#window_copy_synchronize_cursor)  (Implementation)


---
### window_copy_get_selection<!-- {{#callable_declaration:window_copy_get_selection}} -->
Retrieves the current selection from the window copy mode.
- **Description**: This function is used to obtain the text currently selected in the window copy mode. It should be called when the user wants to access the selected text, typically after a selection has been made. The function will return a pointer to the selected text and set the length of the selection in the provided `len` parameter. If no selection is made, it will return NULL and set `len` to 0. The caller is responsible for freeing the returned buffer when it is no longer needed.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. Must not be null.
    - `len`: A pointer to a `size_t` variable where the length of the selection will be stored. The caller must ensure this pointer is valid.
- **Output**: Returns a pointer to a dynamically allocated buffer containing the selected text. The caller must free this buffer when done. If there is no selection, returns NULL.
- **See also**: [`window_copy_get_selection`](#window_copy_get_selection)  (Implementation)


---
### window_copy_copy_buffer<!-- {{#callable_declaration:window_copy_copy_buffer}} -->
Copies data to the clipboard or paste buffer.
- **Description**: This function is used to copy a specified buffer of data to either the clipboard or the paste buffer, depending on the parameters provided. It should be called when you want to transfer data from the current window to the clipboard or paste buffer, typically after a selection has been made. The `set_clip` parameter determines if the clipboard should be set, while `set_paste` indicates if the data should be added to the paste buffer. If the `len` parameter is zero or if `buf` is null, no action is taken.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the current window mode. Must not be null.
    - `prefix`: A string prefix to be added to the copied data. Can be null.
    - `buf`: A pointer to the data buffer to be copied. Must not be null.
    - `len`: The length of the data to be copied. Must be greater than zero.
    - `set_paste`: An integer flag indicating whether to set the paste buffer. A non-zero value indicates to set the paste buffer.
    - `set_clip`: An integer flag indicating whether to set the clipboard. A non-zero value indicates to set the clipboard.
- **Output**: None
- **See also**: [`window_copy_copy_buffer`](#window_copy_copy_buffer)  (Implementation)


---
### window_copy_pipe<!-- {{#callable_declaration:window_copy_pipe}} -->
Copies data from the window to a specified command.
- **Description**: This function is used to copy the contents of a window to a command specified by the user. It should be called when the user wants to send the selected text or data from the window to an external command for processing. The function expects a valid `window_mode_entry` and `session` structure, and a command string that specifies the target command. If the command is null or empty, the default copy command will be used. The function does not return any value.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the current window mode. Must not be null.
    - `s`: A pointer to a `session` structure representing the current session. Must not be null.
    - `cmd`: A string representing the command to which the data will be sent. Can be null or an empty string, in which case the default command will be used.
- **Output**: None
- **See also**: [`window_copy_pipe`](#window_copy_pipe)  (Implementation)


---
### window_copy_copy_pipe<!-- {{#callable_declaration:window_copy_copy_pipe}} -->
Copies data from a window to a specified command.
- **Description**: This function is used to copy data from a window in copy mode to a command specified by the user. It must be called when the window is in copy mode and the session is valid. The function handles the copying process and can set the clipboard or paste buffer based on the provided flags. If the command is invalid or if there is no data to copy, the function will not perform any action.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode. Must not be null.
    - `s`: A pointer to the `session` structure associated with the current session. Must not be null.
    - `prefix`: A string prefix to be used in the copy operation. Can be null.
    - `cmd`: A command string to which the copied data will be sent. Can be null or empty.
    - `set_paste`: An integer flag indicating whether to set the paste buffer. Non-zero values enable this feature.
    - `set_clip`: An integer flag indicating whether to set the clipboard. Non-zero values enable this feature.
- **Output**: The function does not return a value. It performs the copy operation and may modify the clipboard or paste buffer based on the provided flags.
- **See also**: [`window_copy_copy_pipe`](#window_copy_copy_pipe)  (Implementation)


---
### window_copy_copy_selection<!-- {{#callable_declaration:window_copy_copy_selection}} -->
Copies the current selection to the clipboard.
- **Description**: This function is used to copy the selected text from the window copy mode to the clipboard. It should be called when there is a valid selection to copy. The function takes a prefix string that can be used to modify the copied content, and two flags that determine whether to set the clipboard and paste buffers. If the selection is empty or invalid, the function will not perform any action.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` representing the current window mode. Must not be null.
    - `prefix`: A string that is prepended to the copied content. Can be null.
    - `set_paste`: An integer flag indicating whether to set the paste buffer. A value of 1 means to set it, while 0 means not to.
    - `set_clip`: An integer flag indicating whether to set the clipboard. A value of 1 means to set it, while 0 means not to.
- **Output**: None
- **See also**: [`window_copy_copy_selection`](#window_copy_copy_selection)  (Implementation)


---
### window_copy_append_selection<!-- {{#callable_declaration:window_copy_append_selection}} -->
Appends the current selection to the paste buffer.
- **Description**: This function is used to append the currently selected text from the window copy mode to the paste buffer. It should be called when there is a valid selection to be appended. If the clipboard option is enabled, it also updates the clipboard with the current selection. The function handles the case where the paste buffer already contains data by concatenating the new selection to the existing data. If the selection is invalid or empty, the function will simply return without making any changes.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. This must not be null.
- **Output**: Returns nothing. If the selection is valid, it appends the selection to the paste buffer and updates the clipboard if enabled.
- **See also**: [`window_copy_append_selection`](#window_copy_append_selection)  (Implementation)


---
### window_copy_clear_selection<!-- {{#callable_declaration:window_copy_clear_selection}} -->
Clears the current selection in the window copy mode.
- **Description**: This function is used to clear any active text selection in the window copy mode. It resets the cursor drag state and selection flags, ensuring that the selection is completely cleared. It should be called when the user wants to discard the current selection, such as when starting a new selection or exiting the copy mode. The function does not return any value and does not modify any parameters.
- **Inputs**:
    - None
- **Output**: None
- **See also**: [`window_copy_clear_selection`](#window_copy_clear_selection)  (Implementation)


---
### window_copy_copy_line<!-- {{#callable_declaration:window_copy_copy_line}} -->
Copies a specified range of characters from a line in a window.
- **Description**: This function is used to copy a segment of a line from a window's buffer into a provided buffer. It should be called with valid start and end indices that define the range of characters to copy. The function handles wrapped lines appropriately, ensuring that the entire line is copied if it is wrapped. If the specified start index is greater than the end index, the function will return without performing any operation. Additionally, a newline character is appended to the buffer unless the line is wrapped and the end index matches the line's length.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` representing the current window mode. Must not be null.
    - `buf`: A pointer to a character pointer where the copied data will be stored. The caller retains ownership of this buffer.
    - `off`: A pointer to a size_t variable that tracks the current offset in the buffer. Must not be null.
    - `sy`: An unsigned integer representing the line number from which to copy. Must be a valid line number.
    - `sx`: An unsigned integer representing the starting column index for copying. Must be less than or equal to the end index.
    - `ex`: An unsigned integer representing the ending column index for copying. Must be greater than or equal to the starting index.
- **Output**: The function does not return a value but modifies the buffer pointed to by `buf` to include the copied characters and updates the offset pointed to by `off`.
- **See also**: [`window_copy_copy_line`](#window_copy_copy_line)  (Implementation)


---
### window_copy_in_set<!-- {{#callable_declaration:window_copy_in_set}} -->
Checks if a character at a specified position is in a given set.
- **Description**: This function is used to determine if a character located at the specified coordinates (px, py) in the backing grid of a window mode entry is part of a specified set of characters. It is typically called during operations that require checking character membership, such as selection or search functionalities. The function expects valid coordinates within the bounds of the grid, and it will return a non-zero value if the character is found in the set, or zero otherwise.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry`. Must not be null and should be properly initialized.
    - `px`: The x-coordinate in the grid where the character is located. Must be within the width of the grid.
    - `py`: The y-coordinate in the grid where the character is located. Must be within the height of the grid.
    - `set`: A pointer to a null-terminated string representing the set of characters to check against. Must not be null.
- **Output**: Returns a non-zero value if the character at the specified position is found in the set; otherwise, returns zero.
- **See also**: [`window_copy_in_set`](#window_copy_in_set)  (Implementation)


---
### window_copy_find_length<!-- {{#callable_declaration:window_copy_find_length}} -->
Finds the length of a specific line in the window copy mode.
- **Description**: This function is used to determine the length of a line in the window copy mode, identified by the line number provided as an argument. It should be called when the window copy mode is active and the user needs to know the length of a specific line for operations such as scrolling or selection. The function expects a valid `window_mode_entry` structure and a valid line index. If the line index is out of bounds, the behavior is undefined.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the current window copy mode. Must not be null.
    - `py`: An unsigned integer representing the line index for which the length is to be determined. Valid range is from 0 to the total number of lines in the grid. If the value is out of bounds, the behavior is undefined.
- **Output**: Returns an unsigned integer representing the length of the specified line in the window copy mode. If the line does not exist, the return value is not defined.
- **See also**: [`window_copy_find_length`](#window_copy_find_length)  (Implementation)


---
### window_copy_cursor_start_of_line<!-- {{#callable_declaration:window_copy_cursor_start_of_line}} -->
Moves the cursor to the start of the current line.
- **Description**: This function is used to reposition the cursor to the beginning of the current line in a window copy mode. It should be called when the user wants to navigate to the start of the line for editing or selection purposes. The function updates the cursor's position based on the current state of the window copy mode, ensuring that the cursor is correctly placed at the start of the line, taking into account any scrolling or offsets that may have occurred.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry`. This structure must be valid and properly initialized before calling the function. It should not be null.
- **Output**: None
- **See also**: [`window_copy_cursor_start_of_line`](#window_copy_cursor_start_of_line)  (Implementation)


---
### window_copy_cursor_back_to_indentation<!-- {{#callable_declaration:window_copy_cursor_back_to_indentation}} -->
Moves the cursor back to the indentation level.
- **Description**: This function is used to reposition the cursor in a window copy mode to the nearest indentation level to the left of the current cursor position. It should be called when the user wants to quickly navigate to the start of a line or block of code that is indented. The function assumes that the window copy mode is active and that the cursor is currently positioned within a valid range. If the cursor is already at the indentation level, it will remain there. The function does not return any value.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. This pointer must not be null.
- **Output**: None
- **See also**: [`window_copy_cursor_back_to_indentation`](#window_copy_cursor_back_to_indentation)  (Implementation)


---
### window_copy_cursor_end_of_line<!-- {{#callable_declaration:window_copy_cursor_end_of_line}} -->
Moves the cursor to the end of the current line.
- **Description**: This function is used to reposition the cursor to the end of the current line in a copy mode context. It should be called when the user wants to quickly navigate to the end of the line for further editing or selection. The function takes into account whether there is a selection active and adjusts the cursor position accordingly. It is important to note that this function does not return any value and directly modifies the cursor position within the window copy mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode context. This parameter must not be null.
- **Output**: None
- **See also**: [`window_copy_cursor_end_of_line`](#window_copy_cursor_end_of_line)  (Implementation)


---
### window_copy_other_end<!-- {{#callable_declaration:window_copy_other_end}} -->
Copies the selection to the other end.
- **Description**: This function is used to toggle the selection endpoints in a window copy mode. It should be called when there is an active selection, either in a rectangular or line selection mode. The function will switch the selection direction and update the cursor position accordingly. If the selection is currently in the left-to-right mode, it will switch to right-to-left mode and vice versa. The function also handles the cursor drag state, allowing for a more intuitive selection experience.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the current window mode data. Must not be null.
- **Output**: None
- **See also**: [`window_copy_other_end`](#window_copy_other_end)  (Implementation)


---
### window_copy_cursor_left<!-- {{#callable_declaration:window_copy_cursor_left}} -->
Moves the cursor one position to the left.
- **Description**: This function is used to move the cursor left in the window copy mode. It should be called when the user wants to navigate left within the text being copied. The function updates the cursor position and handles any necessary adjustments to ensure the cursor remains within the bounds of the text. It is important to note that this function should be called only when the window is in copy mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. This parameter must not be null.
- **Output**: None
- **See also**: [`window_copy_cursor_left`](#window_copy_cursor_left)  (Implementation)


---
### window_copy_cursor_right<!-- {{#callable_declaration:window_copy_cursor_right}} -->
Moves the cursor one position to the right.
- **Description**: This function is used to move the cursor to the right within a window copy mode. It can be called at any time when the window is in copy mode. The 'all' parameter determines whether the cursor should move past any selected text. If the cursor is at the end of a line, it will wrap to the beginning of the next line.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. Must not be null.
    - `all`: An integer that indicates whether to move the cursor past selected text (1) or not (0). Valid values are 0 or 1.
- **Output**: None
- **See also**: [`window_copy_cursor_right`](#window_copy_cursor_right)  (Implementation)


---
### window_copy_cursor_up<!-- {{#callable_declaration:window_copy_cursor_up}} -->
Moves the cursor up in the window copy mode.
- **Description**: This function is used to move the cursor up within the window copy mode. It can be called in two modes: 'scroll only' or 'normal'. In 'scroll only' mode, the cursor will move up without changing the selection, while in normal mode, it will adjust the selection if applicable. The function should be called when the window copy mode is active, and it handles edge cases such as the cursor being at the top of the screen or when there is a selection active.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. Must not be null.
    - `scroll_only`: An integer flag indicating whether to only scroll without changing the selection. Valid values are 0 (normal mode) and 1 (scroll only).
- **Output**: None
- **See also**: [`window_copy_cursor_up`](#window_copy_cursor_up)  (Implementation)


---
### window_copy_cursor_down<!-- {{#callable_declaration:window_copy_cursor_down}} -->
Moves the cursor down in copy mode.
- **Description**: This function is used to move the cursor down within the copy mode of a window. It can be called in two modes: 'scroll only' or 'normal'. In 'scroll only' mode, if the cursor is at the bottom of the screen, it will scroll the view up without changing the cursor position. In normal mode, the cursor will move down one line unless it is at the bottom of the screen, in which case it will scroll the view up. The function also handles selection updates based on the current selection mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. Must not be null.
    - `scroll_only`: An integer flag indicating whether to only scroll the view (1) or to move the cursor normally (0). Valid values are 0 or 1.
- **Output**: None
- **See also**: [`window_copy_cursor_down`](#window_copy_cursor_down)  (Implementation)


---
### window_copy_cursor_jump<!-- {{#callable_declaration:window_copy_cursor_jump}} -->
Jumps the cursor to the next occurrence of a specified character.
- **Description**: This function is used to move the cursor within a window copy mode to the next occurrence of a specified jump character. It should be called when the user wants to navigate through the text based on specific characters. The function assumes that the window copy mode has been properly initialized and that the cursor is currently positioned within the text. If the jump character is not found, the cursor remains in its current position. The function does not modify any other state or data outside of the cursor position.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. This must not be null.
- **Output**: None
- **See also**: [`window_copy_cursor_jump`](#window_copy_cursor_jump)  (Implementation)


---
### window_copy_cursor_jump_back<!-- {{#callable_declaration:window_copy_cursor_jump_back}} -->
Moves the cursor back to the previous occurrence of a specified character.
- **Description**: This function is intended to be called when the user wants to navigate backward in a copy mode interface to the last occurrence of a specified jump character. It should be invoked when the cursor is already positioned within the copy mode context. The function updates the cursor's position based on the specified jump character, and if the character is found, it will adjust the cursor's coordinates accordingly. If the jump character is not found, the cursor remains in its current position. It is important to ensure that the copy mode is active before calling this function.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode context. This must not be null.
- **Output**: None
- **See also**: [`window_copy_cursor_jump_back`](#window_copy_cursor_jump_back)  (Implementation)


---
### window_copy_cursor_jump_to<!-- {{#callable_declaration:window_copy_cursor_jump_to}} -->
Moves the cursor to the next occurrence of a specified character.
- **Description**: This function is used to navigate within a text buffer in copy mode by jumping the cursor to the next occurrence of a specified character. It should be called when the user wants to quickly locate a character in the text. The function expects that the cursor is already positioned within the text buffer, and it will update the cursor's position based on the specified jump character. If the jump character is not found, the cursor will remain in its current position. It is important to ensure that the function is called in the appropriate context where the text buffer is accessible.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. Must not be null.
- **Output**: None
- **See also**: [`window_copy_cursor_jump_to`](#window_copy_cursor_jump_to)  (Implementation)


---
### window_copy_cursor_jump_to_back<!-- {{#callable_declaration:window_copy_cursor_jump_to_back}} -->
Copies the cursor position to the last jump back position.
- **Description**: This function is used to navigate the cursor back to the last position marked by a specific character during a copy operation. It should be called when the user wants to return to a previous cursor position after making a jump. The function assumes that the window copy mode has been initialized and that the cursor is currently positioned within the bounds of the screen. If the jump character is not found, the cursor position remains unchanged.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry`. This structure must be properly initialized and must not be null.
- **Output**: None
- **See also**: [`window_copy_cursor_jump_to_back`](#window_copy_cursor_jump_to_back)  (Implementation)


---
### window_copy_cursor_next_word<!-- {{#callable_declaration:window_copy_cursor_next_word}} -->
Moves the cursor to the next word.
- **Description**: This function is used to navigate the cursor to the next word in the current window copy mode. It should be called when the user wants to move the cursor forward by one word, which is defined by the provided separators. The function updates the cursor position based on the current cursor location and the specified word separators. It is important to ensure that this function is called in a valid context where the window copy mode is active.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. Must not be null.
    - `separators`: A string containing characters that are considered as word separators. This string must not be null.
- **Output**: None
- **See also**: [`window_copy_cursor_next_word`](#window_copy_cursor_next_word)  (Implementation)


---
### window_copy_cursor_next_word_end_pos<!-- {{#callable_declaration:window_copy_cursor_next_word_end_pos}} -->
Computes the end position of the next word based on the cursor's current position.
- **Description**: This function is used to determine the position of the end of the next word in a text buffer, which is useful for text navigation in a copy mode interface. It should be called when the cursor is positioned within the text, and it will update the provided pointers with the new cursor position. The function takes into account the specified word separators and adjusts the cursor position accordingly. It is important to ensure that the function is called in a valid context where the text buffer is accessible.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure that contains the current state of the window mode. Must not be null.
    - `separators`: A string containing characters that are considered as word separators. This can be any valid string, including whitespace characters. Must not be null.
    - `ppx`: A pointer to an unsigned integer where the x-coordinate of the cursor's new position will be stored. The caller retains ownership.
    - `ppy`: A pointer to an unsigned integer where the y-coordinate of the cursor's new position will be stored. The caller retains ownership.
- **Output**: The function does not return a value. Instead, it updates the values pointed to by `ppx` and `ppy` with the new cursor position after calculating the end of the next word.
- **See also**: [`window_copy_cursor_next_word_end_pos`](#window_copy_cursor_next_word_end_pos)  (Implementation)


---
### window_copy_cursor_next_word_end<!-- {{#callable_declaration:window_copy_cursor_next_word_end}} -->
Moves the cursor to the end of the next word.
- **Description**: This function is used to navigate the cursor to the end of the next word in a window copy mode. It should be called when the user wants to move the cursor forward by one word, which is defined by the provided separators. The function takes into account the current mode of operation (e.g., Vi or Emacs) to determine how to handle whitespace. It is important to ensure that this function is called in a valid context where the window copy mode is active, and the cursor is positioned correctly. If the cursor is already at the end of the line, it will not move beyond that.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. Must not be null.
    - `separators`: A string containing characters that are considered as word separators. This string must not be null.
    - `no_reset`: An integer flag indicating whether to reset the cursor position. This can be 0 or 1.
- **Output**: None
- **See also**: [`window_copy_cursor_next_word_end`](#window_copy_cursor_next_word_end)  (Implementation)


---
### window_copy_cursor_previous_word_pos<!-- {{#callable_declaration:window_copy_cursor_previous_word_pos}} -->
Copies the cursor position to the previous word.
- **Description**: This function is used to move the cursor to the position of the previous word in a text buffer, defined by specified separators. It should be called when the cursor is in a copy mode context, and it updates the provided pointer variables with the new cursor position. The function does not perform any checks for the validity of the input pointers, so they must be valid and point to allocated memory. If the cursor is already at the beginning of the text, it will remain there.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode context. Must not be null.
    - `separators`: A string containing characters that are considered as word separators. Must not be null.
    - `ppx`: A pointer to a `u_int` where the new x-coordinate of the cursor will be stored. Must point to valid memory.
    - `ppy`: A pointer to a `u_int` where the new y-coordinate of the cursor will be stored. Must point to valid memory.
- **Output**: The function does not return a value. Instead, it updates the values pointed to by `ppx` and `ppy` with the new cursor position.
- **See also**: [`window_copy_cursor_previous_word_pos`](#window_copy_cursor_previous_word_pos)  (Implementation)


---
### window_copy_cursor_previous_word<!-- {{#callable_declaration:window_copy_cursor_previous_word}} -->
Moves the cursor to the previous word.
- **Description**: This function is used to navigate the cursor to the beginning of the previous word in a text buffer. It should be called when the user is in copy mode and wishes to move the cursor backward by one word. The function takes into account the specified word separators and the current cursor position. If the cursor is already at the beginning of the line or if there are no previous words, the cursor will remain in its current position. The function does not modify the text buffer.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. Must not be null.
    - `separators`: A string containing characters that are considered as word separators. Must not be null.
    - `already`: An integer indicating whether the cursor has already moved past a word. This can be used to control the behavior of the function. Valid values are typically 0 or 1.
- **Output**: None
- **See also**: [`window_copy_cursor_previous_word`](#window_copy_cursor_previous_word)  (Implementation)


---
### window_copy_cursor_prompt<!-- {{#callable_declaration:window_copy_cursor_prompt}} -->
Copies the cursor prompt in the specified direction.
- **Description**: This function is used to navigate the cursor prompt in a copy mode interface, allowing the user to move the cursor either up or down through the available lines. It should be called when the user wants to change the position of the cursor prompt based on the specified direction. The function takes care of updating the cursor's position and redrawing the screen accordingly. It is important to ensure that the direction parameter is valid and that the function is called in the appropriate context where a cursor prompt is active.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the current window mode. Must not be null.
    - `direction`: An integer indicating the direction to move the cursor. Use 0 for up and 1 for down. Must be either 0 or 1.
    - `start_output`: An integer indicating whether to start output. Use 1 to start output and 0 otherwise.
- **Output**: None
- **See also**: [`window_copy_cursor_prompt`](#window_copy_cursor_prompt)  (Implementation)


---
### window_copy_scroll_up<!-- {{#callable_declaration:window_copy_scroll_up}} -->
Scrolls the content of the window up.
- **Description**: This function is used to scroll the content of a window upwards by a specified number of lines. It should be called when the window is in copy mode, and it adjusts the scroll position based on the current offset. If the specified number of lines to scroll exceeds the current offset, it will limit the scroll to the maximum available offset. The function also updates any search marks if they are present and redraws the selection if necessary.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the current window mode. Must not be null.
    - `ny`: An unsigned integer representing the number of lines to scroll up. Must be greater than zero and less than or equal to the current offset.
- **Output**: None
- **See also**: [`window_copy_scroll_up`](#window_copy_scroll_up)  (Implementation)


---
### window_copy_scroll_down<!-- {{#callable_declaration:window_copy_scroll_down}} -->
Scrolls the content of a window down.
- **Description**: This function is used to scroll the content of a window down by a specified number of lines. It should be called when the window is in copy mode and the user wants to view content that is further down in the buffer. The function ensures that the number of lines to scroll does not exceed the height of the backing screen. If the requested scroll amount exceeds the available content, it adjusts the scroll amount accordingly. The function also updates any search marks if they are present.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode. Must not be null.
    - `ny`: An unsigned integer representing the number of lines to scroll down. Valid values are from 1 to the height of the backing screen. If the value exceeds the available lines, it will be adjusted.
- **Output**: None
- **See also**: [`window_copy_scroll_down`](#window_copy_scroll_down)  (Implementation)


---
### window_copy_rectangle_set<!-- {{#callable_declaration:window_copy_rectangle_set}} -->
Sets the rectangle copy mode for the window.
- **Description**: This function enables or disables rectangle copy mode in the window copy mode. It should be called when the user wants to switch between normal and rectangle selection modes. The function updates the cursor position if it exceeds the current line length, and it also refreshes the selection and redraws the screen to reflect the change in mode. It is important to ensure that this function is called in the appropriate context where the window copy mode is active.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the current window mode. Must not be null.
    - `rectflag`: An integer flag indicating whether to enable (1) or disable (0) rectangle copy mode. Valid values are 0 or 1.
- **Output**: None
- **See also**: [`window_copy_rectangle_set`](#window_copy_rectangle_set)  (Implementation)


---
### window_copy_move_mouse<!-- {{#callable_declaration:window_copy_move_mouse}} -->
Updates the mouse cursor position in the window copy mode.
- **Description**: This function should be called when a mouse event occurs to update the cursor position based on the mouse coordinates. It requires that the window pane is valid and that the current mode is either copy mode or view mode. If the mouse coordinates are outside the bounds of the window pane, the function will not perform any updates. The function also handles the case where the mouse is clicked within the pane, allowing for cursor movement to the clicked position.
- **Inputs**:
    - `m`: A pointer to a `mouse_event` structure that contains the mouse event data. This parameter must not be null.
- **Output**: None
- **See also**: [`window_copy_move_mouse`](#window_copy_move_mouse)  (Implementation)


---
### window_copy_drag_update<!-- {{#callable_declaration:window_copy_drag_update}} -->
Updates the drag state of the window copy mode.
- **Description**: This function is intended to be called during a drag operation in window copy mode. It processes mouse events to update the cursor position and selection state based on the current mouse position. It should be invoked only when the user is actively dragging the mouse to select text. The function handles edge cases such as when the mouse is at the top or bottom of the screen, and it ensures that the selection is visually updated accordingly.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client that initiated the drag. Must not be null.
    - `m`: A pointer to the `mouse_event` structure containing the current mouse event data. Must not be null.
- **Output**: None
- **See also**: [`window_copy_drag_update`](#window_copy_drag_update)  (Implementation)


---
### window_copy_drag_release<!-- {{#callable_declaration:window_copy_drag_release}} -->
Releases the drag operation in window copy mode.
- **Description**: This function should be called when the user releases the mouse button after dragging in window copy mode. It ensures that any ongoing drag operations are properly terminated. The function does not perform any actions if the provided `client` pointer is null or if the mouse event does not correspond to a valid pane. It is important to call this function to maintain the integrity of the drag operation and to prevent any unintended behavior.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client that initiated the drag operation. Must not be null.
    - `m`: A pointer to the `mouse_event` structure containing information about the mouse event. Must not be null.
- **Output**: None
- **See also**: [`window_copy_drag_release`](#window_copy_drag_release)  (Implementation)


---
### window_copy_jump_to_mark<!-- {{#callable_declaration:window_copy_jump_to_mark}} -->
Jumps the cursor to the marked position.
- **Description**: This function is used to move the cursor to a previously marked position within the window copy mode. It updates the cursor's current position to the mark's coordinates, adjusting the view if necessary. It should be called when the user wants to quickly navigate back to a specific point in the text. The function handles the case where the mark is above or below the current view, ensuring that the cursor remains visible after the jump.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the current window mode. Must not be null.
- **Output**: None
- **See also**: [`window_copy_jump_to_mark`](#window_copy_jump_to_mark)  (Implementation)


---
### window_copy_acquire_cursor_up<!-- {{#callable_declaration:window_copy_acquire_cursor_up}} -->
Moves the cursor up in the window copy mode.
- **Description**: This function is used to move the cursor up within the window copy mode. It calculates the new cursor position based on the current cursor position and the height of the window. If the cursor moves beyond the visible area, it will scroll the window up to keep the cursor in view. This function should be called when the user wants to navigate upwards in the text being copied.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode. Must not be null.
    - `hsize`: The height of the window. Must be a positive integer.
    - `oy`: The number of lines scrolled up. Must be a non-negative integer.
    - `oldy`: The previous cursor y-coordinate. Must be a non-negative integer.
    - `px`: The x-coordinate of the cursor. Must be a non-negative integer.
    - `py`: The y-coordinate of the cursor. Must be a non-negative integer.
- **Output**: None
- **See also**: [`window_copy_acquire_cursor_up`](#window_copy_acquire_cursor_up)  (Implementation)


---
### window_copy_acquire_cursor_down<!-- {{#callable_declaration:window_copy_acquire_cursor_down}} -->
Moves the cursor down in the window copy mode.
- **Description**: This function is used to move the cursor down within the window copy mode. It calculates the new cursor position based on the current cursor position and the height of the window. If the cursor moves beyond the visible area, it will scroll the window accordingly. It is important to call this function when the window copy mode is active, and it should be noted that the function handles edge cases where the cursor may exceed the bounds of the screen.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode. Must not be null.
    - `hsize`: An unsigned integer representing the height of the window. Valid values are greater than zero.
    - `sy`: An unsigned integer representing the total number of lines in the window. Must be greater than zero.
    - `oy`: An unsigned integer representing the offset of the window. Must not exceed the height of the window.
    - `oldy`: An unsigned integer representing the previous cursor position. Must not exceed the height of the window.
    - `px`: An unsigned integer representing the x-coordinate of the cursor. Must be within the bounds of the window.
    - `py`: An unsigned integer representing the y-coordinate of the cursor. Must be within the bounds of the window.
    - `no_reset`: An integer flag indicating whether to reset the cursor position. Valid values are 0 (reset) or 1 (do not reset).
- **Output**: None
- **See also**: [`window_copy_acquire_cursor_down`](#window_copy_acquire_cursor_down)  (Implementation)


---
### window_copy_clip_width<!-- {{#callable_declaration:window_copy_clip_width}} -->
Calculates the clipped width for a window copy operation.
- **Description**: This function is used to determine the effective width of a copy operation in a window, ensuring that the total width does not exceed the available space defined by the dimensions of the screen. It should be called when preparing to copy data to ensure that the copied width fits within the specified bounds. The function takes into account the current width, the base offset, and the dimensions of the screen to calculate the appropriate width to use. If the calculated width exceeds the available space, it will return the maximum allowable width.
- **Inputs**:
    - `width`: The desired width to copy. Must be a non-negative integer.
    - `b`: The base offset from which to start copying. Must be a non-negative integer.
    - `sx`: The width of the screen. Must be a positive integer.
    - `sy`: The height of the screen. Must be a positive integer.
- **Output**: Returns the effective width to copy, which is either the original width or the maximum allowable width based on the screen dimensions.
- **See also**: [`window_copy_clip_width`](#window_copy_clip_width)  (Implementation)


---
### window_copy_search_mark_match<!-- {{#callable_declaration:window_copy_search_mark_match}} -->
Marks a match in the search results.
- **Description**: This function is used to mark matches found during a search operation in a copy mode interface. It should be called after a search has been initiated, and it will update the search marks based on the specified parameters. The function takes into account the current position in the grid and the width of the search area, ensuring that the marking does not exceed the grid boundaries. If the marking reaches the maximum value for the search generation, it will wrap around to 1. It is important to note that the function does not perform any operations if the search mark is not found at the specified coordinates.
- **Inputs**:
    - `data`: A pointer to the `window_copy_mode_data` structure that contains the state of the copy mode. Must not be null.
    - `px`: The x-coordinate in the grid where the marking should start. Must be within the grid's width.
    - `py`: The y-coordinate in the grid where the marking should start. Must be within the grid's height.
    - `width`: The width of the area to mark. Must be a positive integer.
    - `regex`: An integer flag indicating whether the search is using regular expressions. Valid values are 0 (false) or 1 (true).
- **Output**: Returns the width of the marked area after the operation, which may be adjusted based on the grid's boundaries.
- **See also**: [`window_copy_search_mark_match`](#window_copy_search_mark_match)  (Implementation)


