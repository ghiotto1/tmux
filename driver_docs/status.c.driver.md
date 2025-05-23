# Purpose
The provided C source code is part of the `tmux` terminal multiplexer, specifically handling the status line and prompt functionalities. This code is responsible for managing the display and interaction of status messages, prompts, and history within the `tmux` environment. It includes functions for loading and saving prompt history, managing status line updates, handling user input for prompts, and providing auto-completion features for command inputs. The code also defines various callback functions and utility functions to support these operations.

Key components of this code include functions for managing the status line's appearance and behavior, such as [`status_redraw`](#status_redraw), [`status_message_set`](#status_message_set), and [`status_prompt_set`](#status_prompt_set). It also includes functions for handling user input and key events, like [`status_prompt_key`](#status_prompt_key), and for managing prompt history with functions like [`status_prompt_load_history`](#status_prompt_load_history) and [`status_prompt_save_history`](#status_prompt_save_history). The code is structured to be integrated into the larger `tmux` application, providing specific functionalities related to user interaction with the status line and command prompts. It does not define a public API but rather serves as an internal component of the `tmux` system.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/time.h`
- `errno.h`
- `limits.h`
- `stdarg.h`
- `stdlib.h`
- `string.h`
- `time.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### status_prompt_find_history_file
- **Type**: `function`
- **Description**: The `status_prompt_find_history_file` function is a static function that determines the file path for the history file used to load or save status prompt history. It checks the 'history-file' option and constructs the path based on the user's home directory if necessary.
- **Use**: This function is used to find the appropriate file path for storing or retrieving status prompt history data.


---
### status_prompt_up_history
- **Type**: `function`
- **Description**: The `status_prompt_up_history` function is a static function that retrieves the previous line from a history list based on the provided index and type. It is used to navigate upwards through a history of prompt inputs.
- **Use**: This function is used to access the previous entry in a history list, typically for user input prompts, by updating the index and returning the corresponding history entry.


---
### status_prompt_down_history
- **Type**: `function`
- **Description**: `status_prompt_down_history` is a static function that retrieves the next line from the history for a given prompt type. It takes two parameters: a pointer to an unsigned integer representing the current index in the history and an unsigned integer representing the prompt type.
- **Use**: This function is used to navigate downwards through the history of user inputs for a specific prompt type, returning the next available history entry.


---
### status_prompt_complete
- **Type**: `function pointer`
- **Description**: `status_prompt_complete` is a static function pointer that takes a `struct client *`, a `const char *`, and a `u_int` as arguments and returns a `char *`. It is used for completing a status prompt in the context of a client.
- **Use**: This function pointer is used to handle the completion logic for status prompts, likely providing suggestions or auto-completions based on the current input.


---
### status_prompt_complete_window_menu
- **Type**: `function pointer`
- **Description**: `status_prompt_complete_window_menu` is a static function that returns a pointer to a character string. It is designed to handle the completion of window menu prompts within a client session, taking parameters such as a client pointer, session pointer, a constant character string, an unsigned integer, and a character.
- **Use**: This function is used to provide completion suggestions for window menu prompts in a client session.


---
### prompt_type_strings
- **Type**: `const char*[]`
- **Description**: `prompt_type_strings` is a static constant array of string literals representing different types of prompts used in the application. The array contains four elements: "command", "search", "target", and "window-target".
- **Use**: This array is used to map prompt types to their corresponding string representations, which can be used for display or logging purposes.


---
### status_prompt_hlist
- **Type**: `char **[PROMPT_NTYPES]`
- **Description**: The `status_prompt_hlist` is a global array of character pointers, where each element is a pointer to a list of strings. The array is indexed by prompt types, as defined by the constant `PROMPT_NTYPES`. This structure is used to store history entries for different types of prompts in the application.
- **Use**: This variable is used to maintain a history of user inputs for different prompt types, allowing the application to retrieve and display past inputs.


---
### status_prompt_hsize
- **Type**: `u_int`
- **Description**: The `status_prompt_hsize` is a global array of unsigned integers with a size defined by the constant `PROMPT_NTYPES`. Each element in this array represents the size of the history list for a specific prompt type, as defined by the `PROMPT_NTYPES` enumeration.
- **Use**: This variable is used to track the number of history entries for each type of prompt in the application.


# Data Structures

---
### status_prompt_menu
- **Type**: `struct`
- **Members**:
    - `c`: A pointer to a client structure associated with the menu.
    - `start`: An unsigned integer indicating the starting index of the menu items.
    - `size`: An unsigned integer representing the number of items in the menu.
    - `list`: A pointer to an array of strings, each representing a menu item.
    - `flag`: A character used as a flag for additional menu options or states.
- **Description**: The `status_prompt_menu` structure is used to define a menu within a status prompt in the tmux application. It holds a reference to a client, the starting index and size of the menu, a list of menu items, and a flag for additional options. This structure is integral to managing and displaying interactive menus in the status line of tmux, allowing users to navigate and select options efficiently.


# Functions

---
### status_prompt_find_history_file<!-- {{#callable:status_prompt_find_history_file}} -->
Finds the history file path for status prompts.
- **Inputs**:
    - `void`: This function does not take any input arguments.
- **Control Flow**:
    - Retrieves the history file name from global options using `options_get_string`.
    - Checks if the retrieved history file name is empty; if so, returns NULL.
    - If the history file name starts with a '/', it duplicates the string and returns it.
    - Checks if the history file name starts with '~/' indicating a home directory reference.
    - If it does, retrieves the home directory path using `find_home`.
    - If the home directory is found, constructs the full path by concatenating the home directory and the history file name, and returns it.
- **Output**:
    - Returns a pointer to the full path of the history file if found, NULL otherwise.


---
### status_prompt_add_typed_history<!-- {{#callable:status_prompt_add_typed_history}} -->
Adds a typed command to the status prompt history.
- **Inputs**:
    - `line`: A string containing the command line input, which may include a type prefix separated by a colon.
- **Control Flow**:
    - The function uses `strsep` to separate the input `line` into a type string (`typestr`) and the rest of the line.
    - If the `line` is not NULL, it determines the prompt type using the [`status_prompt_type`](#status_prompt_type) function.
    - If the determined type is invalid, it handles backward compatibility by appending a colon back to the line and adds the `typestr` to the history as a command type.
    - If the type is valid, it adds the rest of the line to the history with the determined type.
- **Output**:
    - The function does not return a value; it modifies the status prompt history based on the input line.
- **Functions called**:
    - [`status_prompt_type`](#status_prompt_type)
    - [`status_prompt_add_history`](#status_prompt_add_history)


---
### status_prompt_load_history<!-- {{#callable:status_prompt_load_history}} -->
Loads the command history from a specified file into the status prompt.
- **Inputs**:
    - None
- **Control Flow**:
    - Calls [`status_prompt_find_history_file`](#status_prompt_find_history_file) to retrieve the path of the history file.
    - If the history file is not found, the function returns immediately.
    - Attempts to open the history file for reading; if it fails, logs the error and returns.
    - Reads each line from the file in a loop until EOF is reached.
    - For each line, checks if it ends with a newline character and processes it accordingly.
    - If the line ends with a newline, it replaces it with a null terminator and calls [`status_prompt_add_typed_history`](#status_prompt_add_typed_history).
    - If the line does not end with a newline, it allocates memory for the line, copies it, null terminates it, and then calls [`status_prompt_add_typed_history`](#status_prompt_add_typed_history).
    - Finally, closes the file after reading all lines.
- **Output**:
    - The function does not return a value; it populates the command history in the status prompt.
- **Functions called**:
    - [`status_prompt_find_history_file`](#status_prompt_find_history_file)
    - [`status_prompt_add_typed_history`](#status_prompt_add_typed_history)


---
### status_prompt_save_history<!-- {{#callable:status_prompt_save_history}} -->
The `status_prompt_save_history` function saves the current status prompt history to a specified file.
- **Inputs**:
    - None
- **Control Flow**:
    - Calls [`status_prompt_find_history_file`](#status_prompt_find_history_file) to retrieve the path of the history file.
    - If the history file path is NULL, the function returns early.
    - Attempts to open the history file in write mode; if it fails, logs the error and returns.
    - Iterates over all prompt types and their corresponding history entries.
    - Writes each prompt type and its history entry to the file in a formatted manner.
    - Closes the file after writing all entries.
- **Output**:
    - The function does not return a value; it writes the history to a file.
- **Functions called**:
    - [`status_prompt_find_history_file`](#status_prompt_find_history_file)


---
### status_timer_callback<!-- {{#callable:status_timer_callback}} -->
The `status_timer_callback` function manages the status timer for a client, updating the timer based on the session's status interval.
- **Inputs**:
    - `fd`: An unused file descriptor, typically for event handling.
    - `events`: An unused event mask, typically for event handling.
    - `arg`: A pointer to a `client` structure, which contains session and status information.
- **Control Flow**:
    - The function begins by deleting the current status timer for the client.
    - If the session associated with the client is NULL, the function returns immediately.
    - If both `message_string` and `prompt_string` are NULL, it sets the `CLIENT_REDRAWSTATUS` flag for the client.
    - It retrieves the status interval from the session options and sets it in a `timeval` structure.
    - If the status interval is non-zero, it re-adds the timer with the new interval.
    - Finally, it logs the current status interval for debugging purposes.
- **Output**:
    - The function does not return a value; it modifies the state of the client and its timer.


---
### status_timer_start<!-- {{#callable:status_timer_start}} -->
Starts the status timer for a given client.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client for which the status timer is being started.
- **Control Flow**:
    - Checks if the event associated with the client's status timer is already initialized.
    - If initialized, it deletes the existing timer event.
    - If not initialized, it sets the timer event to call [`status_timer_callback`](#status_timer_callback) with the client as an argument.
    - If the session associated with the client is not NULL and the status option is enabled, it immediately calls [`status_timer_callback`](#status_timer_callback).
- **Output**:
    - The function does not return a value; it modifies the state of the client's status timer.
- **Functions called**:
    - [`status_timer_callback`](#status_timer_callback)


---
### status_timer_start_all<!-- {{#callable:status_timer_start_all}} -->
Starts the status timer for all clients.
- **Inputs**:
    - None
- **Control Flow**:
    - Iterates over each `client` in the global `clients` list using `TAILQ_FOREACH`.
    - Calls the [`status_timer_start`](#status_timer_start) function for each `client` to initialize or restart their status timer.
- **Output**:
    - This function does not return a value; it modifies the state of each client by starting their individual status timers.
- **Functions called**:
    - [`status_timer_start`](#status_timer_start)


---
### status_update_cache<!-- {{#callable:status_update_cache}} -->
Updates the status line cache for a given session.
- **Inputs**:
    - `s`: A pointer to a `session` structure that contains the status options.
- **Control Flow**:
    - Retrieves the number of status lines from the session's options using `options_get_number`.
    - If the number of status lines is zero, sets `statusat` to -1, indicating the status line is off.
    - If the status position option is zero, sets `statusat` to 0, indicating the status line is at the top.
    - Otherwise, sets `statusat` to 1, indicating the status line is at the bottom.
- **Output**:
    - The function does not return a value; it modifies the `statuslines` and `statusat` fields of the provided `session` structure.


---
### status_at_line<!-- {{#callable:status_at_line}} -->
The `status_at_line` function calculates the current line number of the status line for a given client.
- **Inputs**:
    - `c`: A pointer to a `struct client` which contains information about the client and its associated session.
- **Control Flow**:
    - The function first retrieves the session associated with the client.
    - It checks if the client's flags indicate that the status line is turned off or if the client is in control mode; if so, it returns -1.
    - Next, it checks the `statusat` field of the session; if it is not equal to 1, it returns the value of `statusat`.
    - If the previous checks are passed, it calculates the line number by subtracting the size of the status line from the total number of lines available in the client's terminal.
- **Output**:
    - The function returns an integer representing the line number of the status line, or -1 if the status line is off.
- **Functions called**:
    - [`status_line_size`](#status_line_size)


---
### status_line_size<!-- {{#callable:status_line_size}} -->
The `status_line_size` function retrieves the size of the status line for a given client.
- **Inputs**:
    - `c`: A pointer to a `struct client` which represents the client whose status line size is being queried.
- **Control Flow**:
    - Checks if the client's flags indicate that the status line is turned off or if the client is in control mode; if so, returns 0.
    - Checks if the session associated with the client is NULL; if it is, retrieves the global status line size from options.
    - If the session is not NULL, returns the status line size specific to that session.
- **Output**:
    - Returns an unsigned integer representing the size of the status line, which is 0 if the status line is off, or the size defined in the session or global options.


---
### status_prompt_line_at<!-- {{#callable:status_prompt_line_at}} -->
The `status_prompt_line_at` function determines the line number for the status prompt in a client's session.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose status prompt line is being queried.
- **Control Flow**:
    - The function retrieves the session associated with the client `c`.
    - It checks if the client has the `CLIENT_STATUSOFF` or `CLIENT_CONTROL` flags set.
    - If either flag is set, it returns 1, indicating the prompt is at the top.
    - If the flags are not set, it retrieves the value of the 'message-line' option from the session's options and returns it.
- **Output**:
    - The function returns an unsigned integer representing the line number for the status prompt, with 1 indicating the top line if the status is off or control mode is active.


---
### status_get_range<!-- {{#callable:status_get_range}} -->
The `status_get_range` function retrieves a `style_range` structure from a specified status line entry based on given x and y coordinates.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client whose status line is being accessed.
    - `x`: An unsigned integer representing the x-coordinate within the status line entry.
    - `y`: An unsigned integer representing the y-coordinate indicating which status line entry to check.
- **Control Flow**:
    - The function first retrieves the `status_line` structure associated with the client `c`.
    - It checks if the provided y-coordinate is within the bounds of the status line entries; if not, it returns NULL.
    - It iterates through the `ranges` list of the specified status line entry at index y.
    - For each `style_range` in the list, it checks if the x-coordinate falls within the start and end of the range.
    - If a matching range is found, it returns a pointer to that `style_range`.
- **Output**:
    - The function returns a pointer to a `style_range` structure if a matching range is found; otherwise, it returns NULL.


---
### status_free_ranges<!-- {{#callable:status_free_ranges}} -->
The `status_free_ranges` function frees all `style_range` structures in a given `style_ranges` list.
- **Inputs**:
    - `srs`: A pointer to a `style_ranges` structure that contains a list of `style_range` elements to be freed.
- **Control Flow**:
    - The function uses `TAILQ_FOREACH_SAFE` to iterate over each `style_range` in the `style_ranges` list.
    - For each `style_range`, it removes the element from the list using `TAILQ_REMOVE`.
    - After removing the element from the list, it frees the memory allocated for that `style_range`.
- **Output**:
    - The function does not return any value; it performs memory deallocation of the `style_range` structures.


---
### status_push_screen<!-- {{#callable:status_push_screen}} -->
The `status_push_screen` function initializes a new status line screen for a client if the current active status line is the default screen.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose status line is being manipulated.
- **Control Flow**:
    - The function retrieves the current status line structure associated with the client.
    - It checks if the current active status line is the default screen.
    - If it is, it allocates memory for a new status line and initializes it with the dimensions of the client's terminal.
    - Finally, it increments the reference count for the status line.
- **Output**:
    - The function does not return a value; it modifies the client's status line structure and manages memory for the status line.
- **Functions called**:
    - [`status_line_size`](#status_line_size)


---
### status_pop_screen<!-- {{#callable:status_pop_screen}} -->
The `status_pop_screen` function manages the reference count of the active status line and frees it if no longer needed.
- **Inputs**:
    - `c`: A pointer to a `struct client` which contains the status line information.
- **Control Flow**:
    - Decrements the reference count of the active status line.
    - If the reference count reaches zero, it frees the currently active screen and its associated memory.
    - Sets the active screen back to the default status line.
- **Output**:
    - The function does not return a value; it modifies the state of the status line associated with the client.


---
### status_init<!-- {{#callable:status_init}} -->
Initializes the status line for a given client.
- **Inputs**:
    - `c`: A pointer to a `struct client` which contains the status line to be initialized.
- **Control Flow**:
    - Accesses the `status_line` structure from the `client` structure.
    - Iterates over the entries in the `status_line` and initializes each entry's `ranges` using `TAILQ_INIT`.
    - Calls `screen_init` to initialize the screen associated with the status line, setting its dimensions based on the client's terminal size.
    - Sets the active status line to the newly initialized screen.
- **Output**:
    - The function does not return a value; it modifies the status line structure of the client directly.


---
### status_free<!-- {{#callable:status_free}} -->
The `status_free` function deallocates resources associated with a client's status line.
- **Inputs**:
    - `c`: A pointer to a `struct client` which contains the status line to be freed.
- **Control Flow**:
    - Accesses the `status` field of the `client` structure to get the `status_line`.
    - Iterates over each entry in the `status_line`'s entries array.
    - Calls [`status_free_ranges`](#status_free_ranges) to free the ranges associated with each entry.
    - Frees the `expanded` field of each entry.
    - Checks if the timer for the status line is initialized and deletes it if so.
    - If the active status line is not the default screen, it frees the active status line.
    - Finally, it frees the default screen status line.
- **Output**:
    - This function does not return a value; it performs cleanup by freeing allocated memory and resources associated with the client's status line.
- **Functions called**:
    - [`status_free_ranges`](#status_free_ranges)


---
### status_redraw<!-- {{#callable:status_redraw}} -->
The `status_redraw` function updates and redraws the status line for a given client.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose status line is to be redrawn.
- **Control Flow**:
    - The function begins by checking if the status line is active; if not, it triggers a fatal error.
    - It retrieves the number of lines for the status line and returns early if there are no lines to display.
    - A format tree is created to handle the formatting of the status line, with flags set based on client options.
    - The function applies default colors for the status line based on session options.
    - It checks if the size of the status line's screen needs to be adjusted and resizes it if necessary.
    - The function then starts writing to the screen context and retrieves the status format options.
    - If no format options are found, it fills the status line with blank spaces.
    - If format options are present, it iterates through each line, expanding the format and checking for changes.
    - If the expanded format differs from the previous one, it updates the status line and marks it as changed.
    - Finally, it stops writing to the screen context, frees the format tree, and returns whether the status line has changed.
- **Output**:
    - The function returns an integer indicating whether the status line has changed (1 if it has changed, 0 otherwise).
- **Functions called**:
    - [`status_line_size`](#status_line_size)
    - [`status_free_ranges`](#status_free_ranges)


---
### status_message_set<!-- {{#callable:status_message_set}} -->
Sets a status message for a client with optional delay and style settings.
- **Inputs**:
    - `struct client *c`: Pointer to the `client` structure representing the client for which the status message is set.
    - `int delay`: The delay in milliseconds before the message is cleared; -1 uses the display-time option, 0 waits for a key press.
    - `int ignore_styles`: Flag indicating whether to ignore style settings for the message.
    - `int ignore_keys`: Flag indicating whether to ignore key inputs while the message is displayed.
    - `int no_freeze`: Flag indicating whether to allow the terminal to be frozen while the message is displayed.
    - `const char *fmt`: Format string for the message to be displayed.
    - `...`: Additional arguments for the format string.
- **Control Flow**:
    - The function starts by initializing a variable argument list and formatting the message string using `xvasprintf`.
    - If the client pointer `c` is NULL, the message is sent to the server and the function returns.
    - The current status message is cleared, and the screen is prepared for the new message.
    - The message string is assigned to the client's message field, and a message is sent to the server indicating the new message.
    - The delay is processed: if it is -1, the display-time option is retrieved; if greater than 0, a timer is set to clear the message after the specified delay.
    - Flags for ignoring keys and styles are set based on the input parameters.
    - If `no_freeze` is not set, the terminal is frozen to prevent user input during the message display.
- **Output**:
    - The function does not return a value; it modifies the state of the client and the server by setting a status message and potentially scheduling its removal.
- **Functions called**:
    - [`status_message_clear`](#status_message_clear)
    - [`status_push_screen`](#status_push_screen)


---
### status_message_clear<!-- {{#callable:status_message_clear}} -->
Clears the status message for a given client.
- **Inputs**:
    - `c`: A pointer to a `struct client` which contains the status message to be cleared.
- **Control Flow**:
    - Checks if the `message_string` of the client is NULL; if it is, the function returns immediately.
    - If `message_string` is not NULL, it frees the memory allocated for it and sets it to NULL.
    - Checks if the `prompt_string` is also NULL; if it is, it clears the cursor and freeze flags from the terminal settings.
    - Sets the `CLIENT_ALLREDRAWFLAGS` in the client's flags to indicate that the status line may need to be redrawn.
    - Calls the [`status_pop_screen`](#status_pop_screen) function to restore the previous status line.
- **Output**:
    - The function does not return a value; it modifies the state of the client by clearing the status message and potentially updating the display.
- **Functions called**:
    - [`status_pop_screen`](#status_pop_screen)


---
### status_message_callback<!-- {{#callable:status_message_callback}} -->
The `status_message_callback` function clears the status message for a client after a specified delay.
- **Inputs**:
    - `fd`: An unused file descriptor, typically for event handling.
    - `event`: An unused event type, indicating the nature of the event.
    - `data`: A pointer to the `client` structure associated with the callback.
- **Control Flow**:
    - The function begins by casting the `data` pointer to a `struct client` type.
    - It then calls the [`status_message_clear`](#status_message_clear) function, passing the client structure to clear any existing status message.
- **Output**:
    - The function does not return a value; it modifies the state of the client by clearing the status message.
- **Functions called**:
    - [`status_message_clear`](#status_message_clear)


---
### status_message_redraw<!-- {{#callable:status_message_redraw}} -->
Redraws the status message on the client's status line.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose status message is to be redrawn.
- **Control Flow**:
    - Checks if the terminal size is valid (non-zero dimensions).
    - Copies the current active screen state to `old_screen` for comparison later.
    - Determines the number of lines for the status line and initializes the screen.
    - Calculates the line number for the message prompt and ensures it does not exceed the available lines.
    - Calculates the length of the message string and limits it to the terminal width.
    - Creates a format tree for styling the message and applies the appropriate style.
    - Starts the screen writing context and copies the existing screen content.
    - Clears the line where the message will be displayed and writes the new message, applying styles if necessary.
    - Stops the screen writing context and compares the new screen with the old screen.
    - If the screens are identical, it frees the old screen and returns 0; otherwise, it frees the old screen and returns 1.
- **Output**:
    - Returns 1 if the status message was changed and redrawn, or 0 if there were no changes.
- **Functions called**:
    - [`status_line_size`](#status_line_size)
    - [`status_prompt_line_at`](#status_prompt_line_at)


---
### status_prompt_accept<!-- {{#callable:status_prompt_accept}} -->
The `status_prompt_accept` function processes the acceptance of a prompt by invoking a callback with a predefined input.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which is unused in this function.
    - `data`: A pointer to a `client` structure that contains the context for the prompt.
- **Control Flow**:
    - The function checks if the `prompt_string` of the `client` is not NULL.
    - If the `prompt_string` is valid, it calls the `prompt_inputcb` callback with the client, prompt data, a string 'y', and a length of 1.
    - After invoking the callback, it calls [`status_prompt_clear`](#status_prompt_clear) to clear the prompt.
    - Finally, it returns `CMD_RETURN_NORMAL` to indicate normal command completion.
- **Output**:
    - The function returns `CMD_RETURN_NORMAL`, indicating that the command was processed successfully.
- **Functions called**:
    - [`status_prompt_clear`](#status_prompt_clear)


---
### status_prompt_set<!-- {{#callable:status_prompt_set}} -->
Sets up a status line prompt for a client with various configurations.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client for which the prompt is being set.
    - `fs`: A pointer to the `cmd_find_state` structure that holds the state of the command being processed.
    - `msg`: A string containing the message to be displayed in the prompt.
    - `input`: A string representing the initial input for the prompt, can be NULL.
    - `inputcb`: A callback function to handle input from the prompt.
    - `freecb`: A callback function to free any associated data when the prompt is cleared.
    - `data`: A pointer to user-defined data that can be passed to the input callback.
    - `flags`: An integer representing various flags that modify the behavior of the prompt.
    - `prompt_type`: An enumeration value indicating the type of prompt being set.
- **Control Flow**:
    - Clears any existing overlay for the client using `server_client_clear_overlay(c)`.
    - Creates a format tree based on the command find state or defaults if `fs` is NULL.
    - If `input` is NULL, it initializes it to an empty string.
    - Clears any existing status messages and prompts, and pushes the current screen state.
    - Sets the prompt formats and message string for the client.
    - Determines the temporary input string based on the `PROMPT_NOFORMAT` flag.
    - Handles the `PROMPT_INCREMENTAL` flag to set the last prompt input and buffer accordingly.
    - Initializes the prompt callbacks and data.
    - Clears the history index and sets the prompt flags and type.
    - Freezes the terminal if the `PROMPT_INCREMENTAL` flag is not set.
    - Redraws the status if necessary and calls the input callback if `PROMPT_INCREMENTAL` is set.
    - Appends a command to accept the prompt if both `PROMPT_SINGLE` and `PROMPT_ACCEPT` flags are set.
- **Output**:
    - The function does not return a value; it modifies the state of the client to display a prompt with the specified configurations.
- **Functions called**:
    - [`status_message_clear`](#status_message_clear)
    - [`status_prompt_clear`](#status_prompt_clear)
    - [`status_push_screen`](#status_push_screen)


---
### status_prompt_clear<!-- {{#callable:status_prompt_clear}} -->
Clears the status prompt for a client.
- **Inputs**:
    - `c`: A pointer to a `struct client` that contains the prompt information to be cleared.
- **Control Flow**:
    - Checks if `c->prompt_string` is NULL; if so, the function returns immediately without doing anything.
    - If `c->prompt_freecb` and `c->prompt_data` are not NULL, it calls the callback function `c->prompt_freecb` with `c->prompt_data` to free any associated data.
    - Frees the memory allocated for `c->prompt_last`, `c->prompt_formats`, `c->prompt_string`, `c->prompt_buffer`, and `c->prompt_saved`, setting them to NULL after freeing.
    - Clears the cursor flags in `c->tty.flags` to allow cursor visibility and unfreeze the terminal.
    - Sets the `CLIENT_ALLREDRAWFLAGS` in `c->flags` to indicate that the client needs to be redrawn.
    - Calls `status_pop_screen(c)` to restore the previous status screen.
- **Output**:
    - The function does not return a value; it modifies the state of the `struct client` by clearing the prompt and freeing associated resources.
- **Functions called**:
    - [`status_pop_screen`](#status_pop_screen)


---
### status_prompt_update<!-- {{#callable:status_prompt_update}} -->
Updates the status prompt for a client with a new message and input.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose prompt is being updated.
    - `msg`: A string containing the new prompt message to be displayed.
    - `input`: A string representing the current input associated with the prompt.
- **Control Flow**:
    - The function begins by freeing the existing prompt string associated with the client.
    - It then duplicates the new message string into the client's prompt string.
    - Next, it frees the existing prompt buffer and formats the input string using `format_expand_time`.
    - The formatted input is converted to UTF-8 and stored in the client's prompt buffer.
    - The prompt index is updated to reflect the length of the new prompt buffer.
    - The history index for the prompt is reset to zero.
    - Finally, the client's flags are updated to indicate that the status needs to be redrawn.
- **Output**:
    - The function does not return a value; it modifies the state of the client by updating its prompt and input buffer.


---
### status_prompt_redraw_character<!-- {{#callable:status_prompt_redraw_character}} -->
The `status_prompt_redraw_character` function redraws a character in the status prompt based on its position and width.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `offset`: An unsigned integer representing the starting offset position for drawing the character.
    - `pwidth`: An unsigned integer indicating the total width of the prompt area.
    - `width`: A pointer to an unsigned integer that tracks the current width of the drawn characters.
    - `gc`: A pointer to a `grid_cell` structure that holds the graphical cell data for the character.
    - `ud`: A pointer to a `utf8_data` structure that contains the UTF-8 encoded character data to be drawn.
- **Control Flow**:
    - Check if the current width is less than the offset; if so, increment the width by the character's width and return 1.
    - If the current width is greater than or equal to the sum of the offset and prompt width, return 0.
    - Increment the width by the character's width and check if it exceeds the prompt width; if so, return 0.
    - Extract the character from the UTF-8 data and check if it is a control character or delete character.
    - If it is a control character, modify the `grid_cell` to represent the control character visually.
    - If it is not a control character, copy the UTF-8 data to the `grid_cell`.
    - Call `screen_write_cell` to draw the character on the screen.
- **Output**:
    - Returns 1 if the drawing can continue, or 0 if it cannot.


---
### status_prompt_redraw_quote<!-- {{#callable:status_prompt_redraw_quote}} -->
Redraws the quote indicator in the status prompt if necessary.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client context.
    - `pcursor`: An unsigned integer representing the position of the cursor in the prompt.
    - `ctx`: A pointer to the `screen_write_ctx` structure used for writing to the screen.
    - `offset`: An unsigned integer representing the offset for drawing.
    - `pwidth`: An unsigned integer representing the width of the prompt.
    - `width`: A pointer to an unsigned integer that will be updated with the current width.
    - `gc`: A pointer to the `grid_cell` structure that defines the graphical cell properties.
- **Control Flow**:
    - Checks if the `PROMPT_QUOTENEXT` flag is set in the client's prompt flags.
    - If the cursor's x-coordinate in the context matches the expected position (pcursor + 1), it prepares to redraw the quote character.
    - Calls the [`status_prompt_redraw_character`](#status_prompt_redraw_character) function to handle the actual drawing of the quote character.
    - If the conditions are not met, it returns 1, indicating that redrawing can continue.
- **Output**:
    - Returns 1 if redrawing can continue, or 0 if it cannot.
- **Functions called**:
    - [`status_prompt_redraw_character`](#status_prompt_redraw_character)


---
### status_prompt_redraw<!-- {{#callable:status_prompt_redraw}} -->
Redraws the status prompt for a client.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the client whose status prompt is to be redrawn.
- **Control Flow**:
    - Checks if the terminal size is valid; if not, returns 0.
    - Copies the current active screen state to `old_screen`.
    - Determines the number of lines for the status line and initializes the screen.
    - Retrieves cursor color and style options from the session's options.
    - Calculates the prompt line position and applies styles based on the prompt mode.
    - Converts the prompt buffer to a string and expands the prompt string with formats.
    - Draws the prompt on the screen, handling cursor positioning and character rendering.
    - Compares the new screen with the old screen; if they are the same, it returns 0, otherwise returns 1.
- **Output**:
    - Returns 1 if the screen has changed and needs to be redrawn, otherwise returns 0.
- **Functions called**:
    - [`status_line_size`](#status_line_size)
    - [`status_prompt_line_at`](#status_prompt_line_at)
    - [`status_prompt_redraw_quote`](#status_prompt_redraw_quote)
    - [`status_prompt_redraw_character`](#status_prompt_redraw_character)


---
### status_prompt_in_list<!-- {{#callable:status_prompt_in_list}} -->
The `status_prompt_in_list` function checks if a character from a given `utf8_data` structure is present in a specified string.
- **Inputs**:
    - `ws`: A pointer to a string where the function will search for a character.
    - `ud`: A pointer to a `utf8_data` structure containing the character to search for.
- **Control Flow**:
    - The function first checks if the size and width of the character in `ud` are both equal to 1.
    - If either condition is not met, the function returns 0, indicating the character is not valid for searching.
    - If the character is valid, it uses `strchr` to check if the character exists in the string `ws`.
    - The function returns 1 if the character is found in `ws`, otherwise it returns 0.
- **Output**:
    - The function returns an integer: 1 if the character is found in the string, and 0 if it is not found or if the character is invalid.


---
### status_prompt_space<!-- {{#callable:status_prompt_space}} -->
The `status_prompt_space` function checks if a given `utf8_data` structure represents a single space character.
- **Inputs**:
    - `ud`: A pointer to a `struct utf8_data` that contains information about a UTF-8 encoded character, including its size, width, and data.
- **Control Flow**:
    - The function first checks if the `size` and `width` fields of the `ud` structure are both equal to 1.
    - If either condition is false, the function returns 0, indicating that the character is not a space.
    - If both conditions are true, it checks if the character data pointed to by `ud->data` is equal to a space character (' ').
    - The function returns 1 if the character is a space, and 0 otherwise.
- **Output**:
    - The function returns an integer: 1 if the character is a space, and 0 if it is not.


---
### status_prompt_translate_key<!-- {{#callable:status_prompt_translate_key}} -->
Translates key inputs for a status prompt based on the current prompt mode.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the current client.
    - `key_code key`: The key code of the key that is being processed.
    - `key_code *new_key`: A pointer to a variable where the translated key code will be stored.
- **Control Flow**:
    - Checks if the client is in `PROMPT_ENTRY` mode.
    - If in `PROMPT_ENTRY`, it processes various key codes and translates them directly or changes the prompt mode.
    - If the key is `Escape`, it switches to `PROMPT_COMMAND` mode and sets a redraw flag.
    - If not in `PROMPT_ENTRY`, it processes other key codes, potentially changing the prompt mode or translating keys.
    - Handles specific key codes like `BSPACE`, `HOME`, `END`, and others to set the new key appropriately.
    - Returns different values based on the processing outcome: 0 to drop the key, 1 to process it, or 2 to append it to the buffer.
- **Output**:
    - Returns an integer indicating the result of the key translation: 0 if the key is dropped, 1 if processed, or 2 if appended.


---
### status_prompt_paste<!-- {{#callable:status_prompt_paste}} -->
The `status_prompt_paste` function pastes text from a clipboard-like buffer into a client's prompt buffer.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the client whose prompt buffer will be modified.
- **Control Flow**:
    - Calculate the current length of the prompt buffer using `utf8_strlen`.
    - Check if there is a saved prompt; if so, use it; otherwise, retrieve the top paste buffer.
    - If the paste buffer is empty, return 0.
    - Iterate through the paste buffer data, converting it to UTF-8 and appending it to a temporary buffer.
    - If the temporary buffer has content, resize the client's prompt buffer to accommodate the new data.
    - Insert the new data into the prompt buffer at the current index, adjusting the index accordingly.
    - Free any allocated memory for the temporary buffer if it was not the saved prompt.
- **Output**:
    - Returns 1 on successful paste operation, or 0 if there was nothing to paste.


---
### status_prompt_replace_complete<!-- {{#callable:status_prompt_replace_complete}} -->
The `status_prompt_replace_complete` function replaces the current word in a prompt with a completed version based on the provided string.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the current client context.
    - `const char *s`: A pointer to a string that represents the completion suggestion; if NULL, the function attempts to generate a completion based on the current word.
- **Control Flow**:
    - The function starts by determining the current cursor position in the prompt and the length of the prompt buffer.
    - It identifies the boundaries of the current word by moving backwards and forwards in the prompt buffer until it finds spaces.
    - If the completion string `s` is NULL, it extracts the current word from the prompt buffer into a temporary array.
    - If `s` is still NULL after this, it attempts to generate a completion suggestion using the [`status_prompt_complete`](#status_prompt_complete) function.
    - The function then calculates the size of the prompt buffer after removing the current word and reallocates the buffer to accommodate the new word.
    - Finally, it inserts the new word into the prompt buffer and updates the cursor position.
- **Output**:
    - The function returns 1 on successful completion of the replacement, or 0 if any step fails (e.g., if the word boundaries are invalid or memory allocation fails).
- **Functions called**:
    - [`status_prompt_space`](#status_prompt_space)
    - [`status_prompt_complete`](#status_prompt_complete)


---
### status_prompt_forward_word<!-- {{#callable:status_prompt_forward_word}} -->
The `status_prompt_forward_word` function moves the prompt index forward to the next word boundary in a command prompt.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure that contains the current prompt state.
    - `size_t size`: The total size of the prompt buffer.
    - `int vi`: An integer flag indicating whether the vi editing mode is active.
    - `const char *separators`: A string containing characters that are considered as word separators.
- **Control Flow**:
    - The function initializes the index `idx` to the current prompt index.
    - If not in vi mode, it skips any leading whitespace characters.
    - If the index is at the end of the prompt buffer, it sets the prompt index to the end and returns.
    - It checks if the current character is a separator or not and sets the `word_is_separators` flag accordingly.
    - The function enters a loop to skip characters until it finds a space or a character that belongs to the opposite class (separator or non-separator).
    - If in vi mode, it skips to the start of the next word if a space is encountered.
- **Output**:
    - The function updates the `prompt_index` in the `client` structure to the new position after moving forward to the next word boundary.
- **Functions called**:
    - [`status_prompt_space`](#status_prompt_space)
    - [`status_prompt_in_list`](#status_prompt_in_list)


---
### status_prompt_end_word<!-- {{#callable:status_prompt_end_word}} -->
The `status_prompt_end_word` function moves the prompt index to the end of the current word in a client's prompt buffer.
- **Inputs**:
    - `c`: A pointer to a `struct client` that contains the prompt buffer and current prompt index.
    - `size`: The total size of the prompt buffer.
    - `separators`: A string containing characters that are considered as word separators.
- **Control Flow**:
    - The function first checks if the current prompt index is at the end of the buffer; if so, it returns immediately.
    - It then increments the index until it finds a non-space character, indicating the start of the next word.
    - Next, it determines if the character at the current index is a separator or not using the [`status_prompt_in_list`](#status_prompt_in_list) function.
    - The function continues to increment the index until it finds a space or a character that is of the opposite class (separator or non-separator).
    - Finally, it sets the prompt index to the last character of the identified word.
- **Output**:
    - The function does not return a value; instead, it updates the prompt index in the `struct client` to point to the end of the current word.
- **Functions called**:
    - [`status_prompt_space`](#status_prompt_space)
    - [`status_prompt_in_list`](#status_prompt_in_list)


---
### status_prompt_backward_word<!-- {{#callable:status_prompt_backward_word}} -->
The `status_prompt_backward_word` function moves the prompt index backward to the beginning of the previous word.
- **Inputs**:
    - `c`: A pointer to the `client` structure that contains the current prompt state.
    - `separators`: A string of characters that are considered as word separators.
- **Control Flow**:
    - The function starts by initializing an index `idx` to the current prompt index of the client.
    - It then enters a loop to decrement `idx` until it finds a non-whitespace character.
    - Next, it checks if the character at the current index is a separator using the [`status_prompt_in_list`](#status_prompt_in_list) function.
    - Another loop is entered to continue decrementing `idx` until it finds a space or a character that is of a different class (separator or non-separator).
    - Finally, it updates the client's prompt index to the calculated `idx`.
- **Output**:
    - The function does not return a value; it modifies the `prompt_index` of the `client` structure to reflect the new position in the prompt buffer.
- **Functions called**:
    - [`status_prompt_space`](#status_prompt_space)
    - [`status_prompt_in_list`](#status_prompt_in_list)


---
### status_prompt_key<!-- {{#callable:status_prompt_key}} -->
Processes key input for a status prompt in a client.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the current client.
    - `key_code key`: The key code representing the key that was pressed.
- **Control Flow**:
    - Checks if the prompt is in key mode and handles the key accordingly.
    - Handles numeric input if the prompt is in numeric mode.
    - Processes various control keys for cursor movement and text manipulation.
    - Handles special keys like Tab for completion and Enter for submitting input.
    - Updates the prompt buffer based on the key pressed and manages the prompt state.
- **Output**:
    - Returns 0 on success, indicating that the key was processed, or 1 if the input was submitted.
- **Functions called**:
    - [`status_prompt_clear`](#status_prompt_clear)
    - [`status_prompt_translate_key`](#status_prompt_translate_key)
    - [`status_prompt_replace_complete`](#status_prompt_replace_complete)
    - [`status_prompt_space`](#status_prompt_space)
    - [`status_prompt_in_list`](#status_prompt_in_list)
    - [`status_prompt_forward_word`](#status_prompt_forward_word)
    - [`status_prompt_end_word`](#status_prompt_end_word)
    - [`status_prompt_backward_word`](#status_prompt_backward_word)
    - [`status_prompt_up_history`](#status_prompt_up_history)
    - [`status_prompt_down_history`](#status_prompt_down_history)
    - [`status_prompt_paste`](#status_prompt_paste)
    - [`status_prompt_add_history`](#status_prompt_add_history)


---
### status_prompt_up_history<!-- {{#callable:status_prompt_up_history}} -->
Retrieves the previous line from the status prompt history.
- **Inputs**:
    - `idx`: A pointer to an array of unsigned integers representing the current index for each prompt type.
    - `type`: An unsigned integer representing the type of prompt for which the history is being accessed.
- **Control Flow**:
    - Checks if the history size for the specified prompt type is zero or if the current index equals the history size, returning NULL if true.
    - Increments the index for the specified prompt type.
    - Returns the history entry corresponding to the updated index.
- **Output**:
    - Returns a pointer to the previous history entry as a string, or NULL if there are no entries available.


---
### status_prompt_down_history<!-- {{#callable:status_prompt_down_history}} -->
Retrieves the previous entry from the status prompt history.
- **Inputs**:
    - `idx`: An array of unsigned integers representing the current index for each prompt type.
    - `type`: An unsigned integer representing the specific type of prompt history to access.
- **Control Flow**:
    - Checks if the history size for the specified type is zero or if the index for that type is zero, returning an empty string if true.
    - Decrements the index for the specified type.
    - If the decremented index is zero, returns an empty string.
    - Returns the history entry corresponding to the decremented index.
- **Output**:
    - Returns a pointer to the previous history entry as a string, or an empty string if there are no entries.


---
### status_prompt_add_history<!-- {{#callable:status_prompt_add_history}} -->
The `status_prompt_add_history` function manages the addition of a new line to the command prompt history, ensuring that duplicates are not added and that the history size does not exceed a specified limit.
- **Inputs**:
    - `line`: A pointer to a string representing the line to be added to the history.
    - `type`: An unsigned integer representing the type of prompt for which the history is being maintained.
- **Control Flow**:
    - The function first retrieves the current size of the history for the specified prompt type.
    - It checks if the line to be added is the same as the last line in the history; if so, it sets a flag to prevent duplication.
    - The function then retrieves the maximum allowed history size from global options.
    - If the current history size is less than the limit, it adds the new line; otherwise, it calculates how many entries need to be freed to maintain the limit.
    - It frees the necessary entries from the history and adjusts the memory allocation for the history list.
    - Finally, if the new line is unique and the new size is greater than zero, it duplicates the line and adds it to the history.
- **Output**:
    - The function does not return a value; it modifies the global history list and size for the specified prompt type.


---
### status_prompt_add_list<!-- {{#callable:status_prompt_add_list}} -->
The `status_prompt_add_list` function adds a unique string to a dynamically allocated list.
- **Inputs**:
    - `list`: A pointer to a pointer to a list of strings, which will be modified to include the new string.
    - `size`: A pointer to an unsigned integer representing the current size of the list.
    - `s`: A constant string that is to be added to the list if it is not already present.
- **Control Flow**:
    - Iterates through the existing list to check if the string `s` is already present.
    - If `s` is found in the list, the function returns early without making any changes.
    - If `s` is not found, the list is reallocated to accommodate one more string.
    - The new string `s` is duplicated and added to the end of the list, and the size is incremented.
- **Output**:
    - The function does not return a value; it modifies the list and size in place.


---
### status_prompt_complete_list<!-- {{#callable:status_prompt_complete_list}} -->
The `status_prompt_complete_list` function generates a list of command completions based on a given input string.
- **Inputs**:
    - `size`: A pointer to an unsigned integer that will hold the size of the completion list.
    - `s`: A constant string representing the input for which completions are being generated.
    - `at_start`: An integer flag indicating whether to return completions that start with the input string.
- **Control Flow**:
    - The function initializes the size of the completion list to zero.
    - It iterates over a predefined command table, checking if each command name or alias starts with the input string, adding matches to the completion list.
    - It retrieves command aliases from global options and checks if they match the input string, adding them to the list if they do.
    - If the `at_start` flag is not set, it further checks a predefined options table and a list of layout names for matches, adding any found to the completion list.
    - Finally, it returns the populated completion list.
- **Output**:
    - The function returns a dynamically allocated array of strings containing the matched command completions.
- **Functions called**:
    - [`status_prompt_add_list`](#status_prompt_add_list)


---
### status_prompt_complete_prefix<!-- {{#callable:status_prompt_complete_prefix}} -->
The `status_prompt_complete_prefix` function finds the longest common prefix among a list of strings.
- **Inputs**:
    - `list`: An array of strings (char**) from which the common prefix is to be determined.
    - `size`: An unsigned integer (u_int) representing the number of strings in the list.
- **Control Flow**:
    - The function first checks if the `list` is NULL or if `size` is 0, returning NULL if either condition is true.
    - It initializes the output string `out` by duplicating the first string in the list.
    - It then iterates through the remaining strings in the list, comparing characters from the end of the current `out` string and the current string in the list.
    - If characters do not match, it truncates `out` at that position.
    - Finally, it returns the longest common prefix found in `out`.
- **Output**:
    - Returns a dynamically allocated string containing the longest common prefix of the input strings, or NULL if no common prefix exists.


---
### status_prompt_menu_callback<!-- {{#callable:status_prompt_menu_callback}} -->
The `status_prompt_menu_callback` function handles the selection of items from a status prompt menu.
- **Inputs**:
    - `menu`: A pointer to a `menu` structure, which represents the menu being interacted with.
    - `idx`: An unsigned integer representing the index of the selected item in the menu.
    - `key`: A `key_code` representing the key pressed by the user.
    - `data`: A pointer to user-defined data, which is expected to be a pointer to a `status_prompt_menu` structure.
- **Control Flow**:
    - The function first checks if the `key` is not `KEYC_NONE`, indicating that a key was pressed.
    - If a key was pressed, it adjusts the `idx` by adding the `start` value from the `status_prompt_menu` structure.
    - It then creates a string `s` based on the selected item from the menu, optionally prepending a flag character if it exists.
    - If the prompt type of the client is `PROMPT_TYPE_WINDOW_TARGET`, it updates the client's prompt buffer with the selected string and sets the redraw flag.
    - If the prompt type is not `PROMPT_TYPE_WINDOW_TARGET`, it attempts to replace the current prompt with the selected string using [`status_prompt_replace_complete`](#status_prompt_replace_complete).
    - Finally, it frees the memory allocated for the menu items and the list.
- **Output**:
    - The function does not return a value; it modifies the state of the client based on the selected menu item and updates the prompt accordingly.
- **Functions called**:
    - [`status_prompt_replace_complete`](#status_prompt_replace_complete)


---
### status_prompt_complete_list_menu<!-- {{#callable:status_prompt_complete_list_menu}} -->
Displays a menu for completing a list of options based on user input.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the user interface context.
    - `list`: An array of strings containing the options to display in the completion menu.
    - `size`: The number of items in the `list` array.
    - `offset`: An integer representing the horizontal offset for displaying the menu.
    - `flag`: A character flag that may modify the behavior of the completion options.
- **Control Flow**:
    - Checks if the size of the list is less than or equal to 1; if so, returns 0 indicating no menu to display.
    - Checks if the available screen space is less than 3 lines; if so, returns 0.
    - Allocates memory for a `status_prompt_menu` structure and initializes it with the provided parameters.
    - Calculates the height of the menu based on available screen space and the number of items.
    - Creates a new menu and populates it with items from the list, assigning keys to each item.
    - Determines the vertical position for displaying the menu based on user preferences.
    - Adjusts the horizontal offset based on the width of the prompt string.
    - Displays the menu using the `menu_display` function, passing the necessary parameters.
    - If the menu display fails, frees allocated resources and returns 0.
    - Returns 1 to indicate successful display of the menu.
- **Output**:
    - Returns 1 if the menu was successfully displayed, otherwise returns 0.
- **Functions called**:
    - [`status_line_size`](#status_line_size)


---
### status_prompt_complete_window_menu<!-- {{#callable:status_prompt_complete_window_menu}} -->
Displays a menu for completing window names based on user input.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the current client.
    - `s`: A pointer to the `session` structure containing the session information.
    - `word`: A string representing the current input word for which completion is requested.
    - `offset`: An unsigned integer representing the current cursor offset in the input.
    - `flag`: A character flag that may modify the behavior of the completion.
- **Control Flow**:
    - Checks if there is enough space on the terminal to display the menu; if not, returns NULL.
    - Allocates memory for a `status_prompt_menu` structure and initializes it with the client and flag.
    - Calculates the maximum height for the menu based on the terminal size and the status line size.
    - Iterates over all window links in the session, filtering them based on the provided word if it is not empty.
    - For each matching window, constructs a display string and adds it to the menu items.
    - If no items are found, frees allocated resources and returns NULL.
    - If only one item is found, returns that item as a string, potentially prepending the flag.
    - If multiple items are found, prepares to display the menu and calls `menu_display` to show it.
    - Handles the case where the menu display fails by freeing resources and returning NULL.
- **Output**:
    - Returns a string representing the selected window name if one is chosen, or NULL if no selection is made or an error occurs.
- **Functions called**:
    - [`status_line_size`](#status_line_size)


---
### status_prompt_complete_sort<!-- {{#callable:status_prompt_complete_sort}} -->
The `status_prompt_complete_sort` function compares two strings for sorting purposes.
- **Inputs**:
    - `a`: A pointer to the first string to compare.
    - `b`: A pointer to the second string to compare.
- **Control Flow**:
    - The function casts the input pointers to `const char **` to access the strings they point to.
    - It then uses the `strcmp` function to compare the two strings pointed to by `aa` and `bb`.
    - The result of `strcmp` is returned, which indicates the lexicographical order of the strings.
- **Output**:
    - The function returns an integer: a negative value if the first string is less than the second, zero if they are equal, and a positive value if the first string is greater than the second.


---
### status_prompt_complete_session<!-- {{#callable:status_prompt_complete_session}} -->
Completes session names or IDs based on a given string.
- **Inputs**:
    - `list`: A pointer to a pointer of character arrays where the completion results will be stored.
    - `size`: A pointer to an unsigned integer that keeps track of the number of completed items.
    - `s`: A constant character pointer representing the input string to match against session names or IDs.
    - `flag`: A character that may be used to modify the output format of the completion results.
- **Control Flow**:
    - Iterates over all sessions using `RB_FOREACH` to check each session's name against the input string.
    - If the input string is empty or matches the beginning of a session name, it reallocates the list and adds the session name to it.
    - If the input string starts with '$', it checks if the subsequent characters match the session ID and adds it to the list if they do.
    - After populating the list, it calls [`status_prompt_complete_prefix`](#status_prompt_complete_prefix) to find the longest common prefix of the completed names.
    - If a flag is provided, it formats the output by prepending the flag to the prefix.
- **Output**:
    - Returns a dynamically allocated string representing the longest common prefix of the completed session names, or NULL if no matches were found.
- **Functions called**:
    - [`status_prompt_complete_prefix`](#status_prompt_complete_prefix)


---
### status_prompt_complete<!-- {{#callable:status_prompt_complete}} -->
The `status_prompt_complete` function provides command completion for a client prompt based on the current input.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client requesting completion.
    - `word`: A pointer to a string containing the current input word for which completion is requested.
    - `offset`: An unsigned integer representing the position of the cursor in the input string.
- **Control Flow**:
    - If the input `word` is empty and the prompt type is not target or window target, return NULL.
    - If the prompt type is not target or window target and the input does not start with '-t' or '-s', call [`status_prompt_complete_list`](#status_prompt_complete_list) to get a list of possible completions.
    - If the prompt type is target or window target, adjust the input string and flag accordingly.
    - If the prompt type is window target, call [`status_prompt_complete_window_menu`](#status_prompt_complete_window_menu) to display a menu for window completion.
    - If the input contains a colon, determine if it is a session completion or a window completion based on the presence of a period after the colon.
    - Sort the completion list if it is not empty and log the results.
    - Return the completed string or NULL if no valid completion is found.
- **Output**:
    - Returns a pointer to a string containing the completed word or NULL if no completion is possible.
- **Functions called**:
    - [`status_prompt_complete_list`](#status_prompt_complete_list)
    - [`status_prompt_complete_prefix`](#status_prompt_complete_prefix)
    - [`status_prompt_complete_window_menu`](#status_prompt_complete_window_menu)
    - [`status_prompt_complete_session`](#status_prompt_complete_session)
    - [`status_prompt_complete_list_menu`](#status_prompt_complete_list_menu)


---
### status_prompt_type<!-- {{#callable:status_prompt_type}} -->
The `status_prompt_type` function maps a string representation of a prompt type to its corresponding enum value.
- **Inputs**:
    - `type`: A constant character pointer representing the string name of the prompt type to be matched.
- **Control Flow**:
    - Iterates through all defined prompt types using a loop indexed by `i`.
    - Compares the input string `type` with each prompt type string using `strcmp`.
    - If a match is found, returns the corresponding enum value `i`.
    - If no match is found after checking all types, returns `PROMPT_TYPE_INVALID`.
- **Output**:
    - Returns an enum value of type `prompt_type` that corresponds to the input string, or `PROMPT_TYPE_INVALID` if no match is found.
- **Functions called**:
    - [`status_prompt_type_string`](#status_prompt_type_string)


---
### status_prompt_type_string<!-- {{#callable:status_prompt_type_string}} -->
Returns the string representation of a prompt type based on its index.
- **Inputs**:
    - `type`: An unsigned integer representing the index of the prompt type.
- **Control Flow**:
    - Checks if the input `type` is greater than or equal to `PROMPT_NTYPES`.
    - If the check is true, returns the string 'invalid'.
    - If the check is false, returns the corresponding string from the `prompt_type_strings` array.
- **Output**:
    - A constant string representing the prompt type or 'invalid' if the input index is out of bounds.


# Function Declarations (Public API)

---
### status_message_callback<!-- {{#callable_declaration:status_message_callback}} -->
Clears the status message for a client.
- **Description**: This function is used to clear any status message that is currently displayed for a client. It should be called when the status message is no longer needed or when it needs to be updated. The function expects a valid client object and will handle the clearing of the message appropriately. It is important to ensure that the client object is valid and properly initialized before calling this function.
- **Inputs**:
    - `fd`: This parameter is unused and can be ignored.
    - `event`: This parameter is unused and can be ignored.
    - `data`: A pointer to a client structure. This must not be null and should point to a valid client object whose status message is to be cleared.
- **Output**: None
- **See also**: [`status_message_callback`](#status_message_callback)  (Implementation)


---
### status_timer_callback<!-- {{#callable_declaration:status_timer_callback}} -->
Handles the status timer event for a client.
- **Description**: This function is a callback that manages the status timer for a client, ensuring that the status is updated at regular intervals as specified by the session's options. It should be used in conjunction with event-driven programming where the status needs to be periodically refreshed. The function checks if the session is valid and updates the client's status flags if necessary. It also resets the timer based on the 'status-interval' option, ensuring that the status is redrawn at the specified interval.
- **Inputs**:
    - `fd`: This parameter is unused and can be ignored.
    - `events`: This parameter is unused and can be ignored.
    - `arg`: A pointer to a 'struct client' object. Must not be null, as it is used to access the client's session and status information.
- **Output**: None
- **See also**: [`status_timer_callback`](#status_timer_callback)  (Implementation)


---
### status_prompt_find_history_file<!-- {{#callable_declaration:status_prompt_find_history_file}} -->
Finds the history file path for status prompts.
- **Description**: This function determines the file path for storing or retrieving status prompt history. It checks the 'history-file' option and returns the appropriate path. If the path is absolute, it duplicates and returns it. If the path starts with '~/,' it expands it using the user's home directory. If the path is empty or invalid, it returns NULL. This function should be used when you need to access the history file for status prompts.
- **Inputs**:
    - None
- **Output**: A dynamically allocated string containing the history file path, or NULL if the path is invalid or cannot be determined.
- **See also**: [`status_prompt_find_history_file`](#status_prompt_find_history_file)  (Implementation)


---
### status_prompt_up_history<!-- {{#callable_declaration:status_prompt_up_history}} -->
Retrieve the previous entry from the history list.
- **Description**: Use this function to navigate upwards through a history list of strings, typically used for command or input history. It should be called when the user wants to access a previous entry in the history. The function requires a valid index pointer and a type identifier to specify which history list to access. It returns NULL if there are no more entries to retrieve, indicating the start of the history list.
- **Inputs**:
    - `idx`: A pointer to an unsigned integer representing the current position in the history list. The value is incremented if a previous entry is available. Must not be null.
    - `type`: An unsigned integer specifying the type of history list to access. Must be within the valid range of history types.
- **Output**: A pointer to a constant character string representing the previous history entry, or NULL if no previous entry exists.
- **See also**: [`status_prompt_up_history`](#status_prompt_up_history)  (Implementation)


---
### status_prompt_down_history<!-- {{#callable_declaration:status_prompt_down_history}} -->
Retrieve the next line from the history for a given type and index.
- **Description**: Use this function to navigate downwards through a history list associated with a specific type. It decrements the index for the specified type and returns the corresponding history entry. If the index is already at the beginning or the history is empty, it returns an empty string. This function is typically used in user interfaces to allow users to cycle through previously entered commands or inputs.
- **Inputs**:
    - `idx`: A pointer to an array of unsigned integers representing the current indices in the history for each type. The caller must ensure this pointer is valid and points to an array with at least as many elements as there are types. The function will modify the index for the specified type.
    - `type`: An unsigned integer representing the type of history to access. This value must be within the valid range of types, as it is used to index into the history arrays. If the type is invalid, behavior is undefined.
- **Output**: Returns a pointer to a constant character string representing the next history entry for the specified type and index. If there is no next entry, it returns an empty string.
- **See also**: [`status_prompt_down_history`](#status_prompt_down_history)  (Implementation)


---
### status_prompt_add_history<!-- {{#callable_declaration:status_prompt_add_history}} -->
Adds a line to the status prompt history.
- **Description**: Use this function to add a new line to the history of a specific prompt type, ensuring that duplicate consecutive entries are not added. The function respects a history limit, removing the oldest entries if necessary to make space for new ones. It is important to ensure that the `type` parameter is within the valid range of prompt types.
- **Inputs**:
    - `line`: A string representing the line to be added to the history. It must not be null, and the caller retains ownership of the string.
    - `type`: An unsigned integer representing the type of prompt history to which the line should be added. It must be within the valid range of prompt types; otherwise, behavior is undefined.
- **Output**: None
- **See also**: [`status_prompt_add_history`](#status_prompt_add_history)  (Implementation)


---
### status_prompt_complete<!-- {{#callable_declaration:status_prompt_complete}} -->
Completes a word based on the current prompt type and input.
- **Description**: This function is used to perform word completion in a command-line interface, based on the current prompt type and the input word. It is typically called when a user requests auto-completion, such as by pressing the Tab key. The function handles different prompt types, including target and window-target prompts, and can complete session names, window names, or other command-related words. It returns a completed word or a list of possible completions, depending on the context. The function must be called with a valid client structure and a non-null word string.
- **Inputs**:
    - `c`: A pointer to a client structure representing the current client session. Must not be null.
    - `word`: A constant character pointer representing the word to be completed. Must not be null.
    - `offset`: An unsigned integer representing the position in the word from which to start completion. Typically zero for full word completion.
- **Output**: A dynamically allocated string containing the completed word or a list of possible completions, or NULL if no completion is possible. The caller is responsible for freeing the returned string.
- **See also**: [`status_prompt_complete`](#status_prompt_complete)  (Implementation)


---
### status_prompt_complete_window_menu<!-- {{#callable_declaration:status_prompt_complete_window_menu}} -->
Displays a window selection menu for command completion.
- **Description**: This function is used to display a menu of windows for command completion in a client session. It is typically called when a user is prompted to select a window from a list, and it provides a menu interface for this selection. The function requires a valid client and session, and it expects the terminal to have enough space to display the menu. If the terminal does not have sufficient space, or if no matching windows are found, the function returns NULL. The function handles both single and multiple window matches, returning the appropriate window identifier or displaying a menu for selection.
- **Inputs**:
    - `c`: A pointer to a client structure representing the current client. Must not be null.
    - `s`: A pointer to a session structure representing the current session. Must not be null.
    - `word`: A string representing the current input word for completion. Can be null or empty.
    - `offset`: An unsigned integer representing the offset for menu positioning. No specific range is required.
    - `flag`: A character flag that modifies the behavior of the completion. Can be any character, including '\0' for no flag.
- **Output**: Returns a dynamically allocated string representing the selected window identifier if a single match is found, or NULL if no match is found or a menu is displayed.
- **See also**: [`status_prompt_complete_window_menu`](#status_prompt_complete_window_menu)  (Implementation)


