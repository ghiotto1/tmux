# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file primarily deals with the management and display of status messages and prompts within the tmux interface. It includes functions for handling status line updates, managing prompt history, and processing user input for command prompts. The code provides a range of functionalities, including loading and saving prompt history, handling user input for prompts, and managing the display of status messages and prompts on the terminal.

Key technical components of the file include functions for managing the status line's appearance and behavior, such as [`status_redraw`](#status_redraw), [`status_message_set`](#status_message_set), and [`status_prompt_set`](#status_prompt_set). The file also defines several static functions for internal operations, like [`status_prompt_find_history_file`](#status_prompt_find_history_file) and [`status_prompt_complete`](#status_prompt_complete), which handle history file management and command completion, respectively. The code is structured to be part of a larger application, with functions that interact with other components of tmux, such as client and session structures. It does not define a public API or external interfaces but rather contributes to the internal workings of tmux by managing user interactions through the status line and command prompts.
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
- **Description**: The `status_prompt_find_history_file` is a static function that determines the file path for loading or saving the status prompt history. It constructs the path based on the 'history-file' option from global options, handling cases where the path is absolute or relative to the user's home directory.
- **Use**: This function is used to locate the history file for status prompts, which is essential for loading and saving command history.


---
### status_prompt_up_history
- **Type**: `function`
- **Description**: The `status_prompt_up_history` function is a static function that retrieves the previous line from a history list based on the provided index and type. It is used to navigate upwards through the history of status prompts.
- **Use**: This function is used to access the previous entry in a history list for status prompts, allowing users to cycle through past inputs.


---
### status_prompt_down_history
- **Type**: `function`
- **Description**: `status_prompt_down_history` is a static function that retrieves the next line from the history for a given prompt type. It takes two parameters: a pointer to an unsigned integer representing the current index in the history and an unsigned integer representing the prompt type.
- **Use**: This function is used to navigate downwards through the history of user inputs for a specific prompt type, returning the next available history entry.


---
### status_prompt_complete
- **Type**: `function`
- **Description**: `status_prompt_complete` is a static function that is used to complete a word or command based on the current input in a prompt. It takes a client, a word, and an offset as parameters and returns a completed string or NULL if no completion is found.
- **Use**: This function is used to provide auto-completion suggestions for user input in a prompt, enhancing user experience by predicting and completing commands or words.


---
### status_prompt_complete_window_menu
- **Type**: `function pointer`
- **Description**: `status_prompt_complete_window_menu` is a static function that returns a pointer to a character. It is used to display a menu for window completion in a status prompt, taking parameters such as a client, session, word, offset, and flag.
- **Use**: This function is used to facilitate window completion by displaying a menu in the status prompt.


---
### prompt_type_strings
- **Type**: `const char*[]`
- **Description**: The `prompt_type_strings` is a static constant array of string literals in C, which contains different types of prompt identifiers used in the application. These strings represent various prompt types such as 'command', 'search', 'target', and 'window-target'. This array is defined with a fixed set of strings and is not intended to be modified at runtime.
- **Use**: This variable is used to map prompt types to their corresponding string representations for display or processing purposes in the application.


---
### status_prompt_hlist
- **Type**: `char **[PROMPT_NTYPES]`
- **Description**: The `status_prompt_hlist` is a global array of character pointers, where each element is a list of strings representing the history of status prompts for different prompt types. The array is indexed by the prompt type, which is defined by the constant `PROMPT_NTYPES`. Each element in the array is a pointer to a list of strings, with each string representing a historical entry for a specific prompt type.
- **Use**: This variable is used to store and manage the history of status prompts for different types, allowing retrieval and manipulation of past prompt entries.


---
### status_prompt_hsize
- **Type**: `u_int`
- **Description**: The `status_prompt_hsize` is a global array of unsigned integers with a size defined by the constant `PROMPT_NTYPES`. Each element in this array represents the size of the history list for a specific type of prompt, as indicated by the index corresponding to different prompt types.
- **Use**: This variable is used to track the number of history entries for each prompt type, allowing the program to manage and access the history of user inputs for different prompts.


# Data Structures

---
### status_prompt_menu
- **Type**: `struct`
- **Members**:
    - `c`: A pointer to a client structure.
    - `start`: An unsigned integer indicating the start index.
    - `size`: An unsigned integer representing the size of the list.
    - `list`: A pointer to an array of strings.
    - `flag`: A character flag used for additional options or states.
- **Description**: The `status_prompt_menu` structure is used to manage a menu prompt within a client session. It holds a reference to the client, the starting index and size of a list of options, and a flag character for additional control. This structure is likely used to facilitate user interaction with a menu, allowing for dynamic updates and navigation through a list of options.


# Functions

---
### status_prompt_find_history_file<!-- {{#callable:status_prompt_find_history_file}} -->
Finds the history file path for the status prompt.
- **Inputs**:
    - `void`: The function does not take any input parameters.
- **Control Flow**:
    - Retrieves the history file name from global options using [`options_get_string`](options.c.driver.md#options_get_string).
    - Checks if the retrieved history file name is empty; if so, returns NULL.
    - If the history file name is an absolute path (starts with '/'), duplicates and returns it.
    - Checks if the history file name starts with '~/' indicating a home directory reference.
    - If it does, retrieves the home directory path using [`find_home`](tmux.c.driver.md#find_home).
    - If the home directory is found, constructs the full path by concatenating the home directory and the history file name (excluding the '~').
    - Returns the constructed path.
- **Output**:
    - Returns a pointer to a string containing the full path to the history file, or NULL if the history file cannot be determined.
- **Functions called**:
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`find_home`](tmux.c.driver.md#find_home)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### status_prompt_add_typed_history<!-- {{#callable:status_prompt_add_typed_history}} -->
Adds a typed command to the status prompt history.
- **Inputs**:
    - `line`: A string representing the command line input, which may include a type prefix followed by a colon.
- **Control Flow**:
    - The function uses [`strsep`](compat/strsep.c.driver.md#strsep) to separate the input `line` into a type string (`typestr`) and the rest of the line.
    - If the remaining line is not NULL, it determines the prompt type using [`status_prompt_type`](#status_prompt_type).
    - If the type is invalid, it adds the `typestr` to the history with a default type of `PROMPT_TYPE_COMMAND`.
    - If the type is valid, it adds the remaining line to the history with the determined type.
- **Output**:
    - The function does not return a value; it modifies the status prompt history based on the input line.
- **Functions called**:
    - [`strsep`](compat/strsep.c.driver.md#strsep)
    - [`status_prompt_type`](#status_prompt_type)
    - [`status_prompt_add_history`](#status_prompt_add_history)


---
### status_prompt_load_history<!-- {{#callable:status_prompt_load_history}} -->
Loads the command history from a specified file into the status prompt.
- **Inputs**:
    - None
- **Control Flow**:
    - Calls `status_prompt_find_history_file()` to retrieve the path of the history file.
    - If the history file is not found, the function returns immediately.
    - Attempts to open the history file for reading; if it fails, logs the error and returns.
    - Reads each line from the file in a loop until EOF is reached.
    - For each line, it checks if it ends with a newline character and processes it accordingly.
    - If the line ends with a newline, it replaces it with a null terminator and calls `status_prompt_add_typed_history()`.
    - If the line does not end with a newline, it allocates memory for the line, copies it, null terminates it, and then calls `status_prompt_add_typed_history()`.
    - Finally, it closes the file after reading all lines.
- **Output**:
    - The function does not return a value; it populates the command history in the status prompt from the loaded file.
- **Functions called**:
    - [`status_prompt_find_history_file`](#status_prompt_find_history_file)
    - [`fgetln`](compat/fgetln.c.driver.md#fgetln)
    - [`status_prompt_add_typed_history`](#status_prompt_add_typed_history)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)


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
    - The function does not return a value; it writes the prompt history to a file.
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
    - It checks if the session associated with the client is NULL; if so, it returns immediately.
    - If both `message_string` and `prompt_string` are NULL, it sets the `CLIENT_REDRAWSTATUS` flag for the client.
    - It retrieves the status interval from the session options and sets it in a `timeval` structure.
    - If the status interval is non-zero, it re-adds the timer with the new interval.
    - Finally, it logs the current status interval for debugging purposes.
- **Output**:
    - The function does not return a value; it modifies the state of the client and its timer based on the session's status interval.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)


---
### status_timer_start<!-- {{#callable:status_timer_start}} -->
Starts the status timer for a given client.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client for which the status timer is being started.
- **Control Flow**:
    - Checks if the event associated with the client's status timer is already initialized.
    - If initialized, it deletes the existing timer event.
    - If not initialized, it sets the timer event to call [`status_timer_callback`](#status_timer_callback) with the client as an argument.
    - Checks if the session associated with the client is not NULL and if the status option is enabled.
    - If both conditions are met, it immediately calls [`status_timer_callback`](#status_timer_callback) with specific parameters.
- **Output**:
    - The function does not return a value; it modifies the state of the client's status timer and may trigger a callback.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
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
    - `s`: A pointer to a `struct session` that contains the session options and status information.
- **Control Flow**:
    - Retrieves the number of status lines from the session options using [`options_get_number`](options.c.driver.md#options_get_number).
    - If the number of status lines is zero, sets `s->statusat` to -1, indicating the status line is off.
    - If the status position option is zero, sets `s->statusat` to 0, indicating the status line is at the top.
    - Otherwise, sets `s->statusat` to 1, indicating the status line is at the bottom.
- **Output**:
    - The function does not return a value; it modifies the `statuslines` and `statusat` fields of the provided session structure.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)


---
### status_at_line<!-- {{#callable:status_at_line}} -->
The `status_at_line` function calculates the current line number of the status line for a given client.
- **Inputs**:
    - `c`: A pointer to a `struct client` which contains information about the client and its associated session.
- **Control Flow**:
    - The function first retrieves the session associated with the client.
    - It checks if the client's flags indicate that the status line is turned off or if the client is in control mode; if so, it returns -1.
    - Next, it checks the `statusat` field of the session; if it is not equal to 1, it returns the value of `statusat`.
    - If the status line is active, it calculates the current line number by subtracting the size of the status line from the total number of lines available in the client's terminal.
- **Output**:
    - The function returns an integer representing the line number of the status line, or -1 if the status line is off.
- **Functions called**:
    - [`status_line_size`](#status_line_size)


---
### status_line_size<!-- {{#callable:status_line_size}} -->
The `status_line_size` function retrieves the size of the status line for a given client.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client for which the status line size is being queried.
- **Control Flow**:
    - The function first checks if the client's flags indicate that the status line is turned off or if the client is in control mode; if so, it returns 0.
    - Next, it checks if the session associated with the client is NULL; if it is, it retrieves the global status line size from options.
    - If the session is not NULL, it returns the `statuslines` value from the session structure.
- **Output**:
    - The function returns an unsigned integer representing the size of the status line, which can be 0 if the status line is off or the size defined in the session or global options.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)


---
### status_prompt_line_at<!-- {{#callable:status_prompt_line_at}} -->
The `status_prompt_line_at` function determines the line number for displaying the status prompt in a client's session.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose status prompt line is being queried.
- **Control Flow**:
    - The function retrieves the session associated with the client `c`.
    - It checks if the client has the `CLIENT_STATUSOFF` or `CLIENT_CONTROL` flags set.
    - If either flag is set, it returns 1, indicating the prompt is at the top.
    - If the flags are not set, it retrieves the value of the `message-line` option from the session's options and returns it.
- **Output**:
    - The function returns an unsigned integer representing the line number for the status prompt, with 1 indicating the top line if the status is off or control mode is active, or the value of the `message-line` option otherwise.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)


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
    - It iterates through the `ranges` linked list of the specified status line entry (at index y).
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
    - Iterates over each `style_range` in the `style_ranges` list using `TAILQ_FOREACH_SAFE` to safely remove and free each element.
    - For each `style_range`, it removes the element from the list and deallocates its memory using `free`.
- **Output**:
    - This function does not return a value; it performs memory deallocation of the `style_range` elements.


---
### status_push_screen<!-- {{#callable:status_push_screen}} -->
The `status_push_screen` function initializes a new status line screen for a client if the current active status line is the default screen.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose status line is being manipulated.
- **Control Flow**:
    - The function retrieves the current status line structure from the client.
    - It checks if the current active status line is the default screen.
    - If it is, it allocates memory for a new status line screen and initializes it with the client's terminal width and the size of the status line.
    - Finally, it increments the reference count for the status line.
- **Output**:
    - The function does not return a value; it modifies the client's status line structure in place.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`screen_init`](screen.c.driver.md#screen_init)
    - [`status_line_size`](#status_line_size)


---
### status_pop_screen<!-- {{#callable:status_pop_screen}} -->
The `status_pop_screen` function manages the reference count of the active status line and frees it if no references remain.
- **Inputs**:
    - `c`: A pointer to a `struct client` which contains the status line information.
- **Control Flow**:
    - Decrements the reference count of the active status line.
    - If the reference count reaches zero, it frees the active status line and resets it to point to the default screen.
- **Output**:
    - The function does not return a value; it modifies the state of the status line associated with the client.
- **Functions called**:
    - [`screen_free`](screen.c.driver.md#screen_free)


---
### status_init<!-- {{#callable:status_init}} -->
Initializes the status line for a given client.
- **Inputs**:
    - `c`: A pointer to a `struct client` that contains the status line to be initialized.
- **Control Flow**:
    - Accesses the `status_line` structure from the `client` structure.
    - Iterates over the entries in the `status_line` and initializes each entry's `ranges` using `TAILQ_INIT`.
    - Calls [`screen_init`](screen.c.driver.md#screen_init) to initialize the screen associated with the status line, setting its dimensions based on the client's terminal size.
    - Sets the active status line to the newly initialized screen.
- **Output**:
    - The function does not return a value; it modifies the status line structure of the client to prepare it for use.
- **Functions called**:
    - [`screen_init`](screen.c.driver.md#screen_init)


---
### status_free<!-- {{#callable:status_free}} -->
The `status_free` function deallocates memory associated with a client's status line.
- **Inputs**:
    - `c`: A pointer to a `struct client` that contains the status line to be freed.
- **Control Flow**:
    - Iterates over each entry in the `status_line` structure associated with the client.
    - Calls [`status_free_ranges`](#status_free_ranges) to free the ranges associated with each entry.
    - Frees the `expanded` string for each entry.
    - Checks if the timer for the status line is initialized and deletes it if so.
    - If the active status line is not the default screen, it frees the active status line.
    - Finally, it frees the default screen status line.
- **Output**:
    - The function does not return a value; it performs memory deallocation to prevent memory leaks.
- **Functions called**:
    - [`status_free_ranges`](#status_free_ranges)
    - [`screen_free`](screen.c.driver.md#screen_free)


---
### status_redraw<!-- {{#callable:status_redraw}} -->
The `status_redraw` function updates and redraws the status line for a given client.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose status line is to be redrawn.
- **Control Flow**:
    - The function begins by checking if the status line is active; if not, it triggers a fatal error.
    - It checks if the terminal's height is zero or if the status line size is zero, returning early if true.
    - A format tree is created for the status line, applying default styles and colors.
    - The function checks if the status line's dimensions need to be resized and performs the resizing if necessary.
    - It starts writing to the screen context and retrieves the status format options.
    - If no format options are found, it fills the status line with spaces; otherwise, it processes each line of the status format.
    - For each line, it expands the format string and checks if it has changed; if so, it updates the status line.
    - Finally, it stops writing to the screen context and frees the format tree before returning whether the status line has changed.
- **Output**:
    - The function returns an integer indicating whether the status line has changed (1) or not (0).
- **Functions called**:
    - [`status_line_size`](#status_line_size)
    - [`format_create`](format.c.driver.md#format_create)
    - [`format_defaults`](format.c.driver.md#format_defaults)
    - [`style_apply`](style.c.driver.md#style_apply)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`grid_cells_equal`](grid.c.driver.md#grid_cells_equal)
    - [`screen_resize`](screen.c.driver.md#screen_resize)
    - [`screen_write_start`](screen-write.c.driver.md#screen_write_start)
    - [`options_get`](options.c.driver.md#options_get)
    - [`screen_write_putc`](screen-write.c.driver.md#screen_write_putc)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`options_array_get`](options.c.driver.md#options_array_get)
    - [`format_expand_time`](format.c.driver.md#format_expand_time)
    - [`status_free_ranges`](#status_free_ranges)
    - [`format_draw`](format-draw.c.driver.md#format_draw)
    - [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop)
    - [`format_free`](format.c.driver.md#format_free)


---
### status_message_set<!-- {{#callable:status_message_set}} -->
Sets a status message for a client with optional delay and style handling.
- **Inputs**:
    - `struct client *c`: Pointer to the client structure that will display the status message.
    - `int delay`: The delay in milliseconds before the message disappears; -1 uses the display-time option, 0 waits for a key press.
    - `int ignore_styles`: Flag indicating whether to ignore style settings for the message.
    - `int ignore_keys`: Flag indicating whether to ignore key inputs while the message is displayed.
    - `int no_freeze`: Flag indicating whether to allow the terminal to be unresponsive (not frozen) while the message is displayed.
    - `const char *fmt`: Format string for the message, followed by variable arguments.
- **Control Flow**:
    - The function starts by initializing a variable argument list and formatting the message string using [`xvasprintf`](xmalloc.c.driver.md#xvasprintf).
    - If the client pointer `c` is NULL, the message is sent to the server log and the function returns.
    - If `c` is not NULL, the current status message is cleared, and the new message is set.
    - The function checks the `delay` parameter to determine how long to display the message, setting up a timer if necessary.
    - Flags for ignoring keys and styles are set based on the input parameters.
    - If `no_freeze` is not set, the terminal is marked as frozen, and the cursor is hidden.
- **Output**:
    - The function does not return a value; it modifies the state of the client and the status message displayed.
- **Functions called**:
    - [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)
    - [`status_message_clear`](#status_message_clear)
    - [`status_push_screen`](#status_push_screen)
    - [`options_get_number`](options.c.driver.md#options_get_number)


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
The `status_message_callback` function clears the status message for a given client after a specified delay.
- **Inputs**:
    - `fd`: An unused file descriptor, typically for event handling.
    - `event`: An unused event type, indicating the event that triggered the callback.
    - `data`: A pointer to the `client` structure associated with the callback.
- **Control Flow**:
    - The function retrieves the `client` structure from the `data` pointer.
    - It calls the [`status_message_clear`](#status_message_clear) function, passing the `client` structure to clear the status message.
- **Output**:
    - The function does not return a value; it modifies the state of the `client` by clearing its status message.
- **Functions called**:
    - [`status_message_clear`](#status_message_clear)


---
### status_message_redraw<!-- {{#callable:status_message_redraw}} -->
The `status_message_redraw` function updates and redraws the status message on the client's status line.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose status message is to be redrawn.
- **Control Flow**:
    - Checks if the terminal size is valid (non-zero width and height).
    - Copies the current active screen state to `old_screen` for comparison later.
    - Determines the number of lines for the status line and initializes the screen.
    - Calculates the line number for the message prompt and ensures it does not exceed the available lines.
    - Calculates the length of the message string and limits it to the terminal width.
    - Creates a format tree for styling the message and applies the appropriate style.
    - Starts the screen writing context and copies the existing screen content.
    - Clears the line where the message will be displayed and writes the new message, applying styles if not ignored.
    - Stops the screen writing context and compares the new screen grid with the old one to check for changes.
- **Output**:
    - Returns 1 if the status message was changed and redrawn, or 0 if there were no changes.
- **Functions called**:
    - [`status_line_size`](#status_line_size)
    - [`screen_init`](screen.c.driver.md#screen_init)
    - [`status_prompt_line_at`](#status_prompt_line_at)
    - [`format_create_defaults`](format.c.driver.md#format_create_defaults)
    - [`style_apply`](style.c.driver.md#style_apply)
    - [`format_free`](format.c.driver.md#format_free)
    - [`screen_write_start`](screen-write.c.driver.md#screen_write_start)
    - [`screen_write_fast_copy`](screen-write.c.driver.md#screen_write_fast_copy)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`screen_write_putc`](screen-write.c.driver.md#screen_write_putc)
    - [`format_draw`](format-draw.c.driver.md#format_draw)
    - [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop)
    - [`grid_compare`](grid.c.driver.md#grid_compare)
    - [`screen_free`](screen.c.driver.md#screen_free)


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
Sets up a status prompt for a client with specified parameters.
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
    - Determines the prompt buffer based on the `flags` provided, either using the input directly or expanding it using the format tree.
    - Handles incremental input by setting the last prompt input and initializing the prompt buffer.
    - Sets the input and free callback functions along with user data.
    - Initializes the prompt history index and sets the prompt flags and type.
    - Freezes the terminal if the prompt is not incremental.
    - Redraws the client status if necessary and invokes the input callback if the prompt is incremental.
    - Appends a command to accept the prompt if both SINGLE and ACCEPT flags are set.
- **Output**:
    - The function does not return a value; it modifies the state of the client to display a prompt with the specified parameters.
- **Functions called**:
    - [`server_client_clear_overlay`](server-client.c.driver.md#server_client_clear_overlay)
    - [`format_create_from_state`](format.c.driver.md#format_create_from_state)
    - [`format_create_defaults`](format.c.driver.md#format_create_defaults)
    - [`status_message_clear`](#status_message_clear)
    - [`status_prompt_clear`](#status_prompt_clear)
    - [`status_push_screen`](#status_push_screen)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`format_expand_time`](format.c.driver.md#format_expand_time)
    - [`utf8_fromcstr`](utf8.c.driver.md#utf8_fromcstr)
    - [`utf8_strlen`](utf8.c.driver.md#utf8_strlen)
    - [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append)


---
### status_prompt_clear<!-- {{#callable:status_prompt_clear}} -->
Clears the status prompt for a client, freeing associated resources.
- **Inputs**:
    - `c`: A pointer to a `struct client` that represents the client whose status prompt is to be cleared.
- **Control Flow**:
    - Checks if the `prompt_string` of the client is NULL; if so, the function returns immediately.
    - If a `prompt_freecb` callback is set and `prompt_data` is not NULL, it calls the callback to free the prompt data.
    - Frees the memory allocated for `prompt_last`, `prompt_formats`, `prompt_string`, `prompt_buffer`, and `prompt_saved`.
    - Clears the cursor flags in the `tty` structure of the client.
    - Sets the `CLIENT_ALLREDRAWFLAGS` in the client's flags to indicate that the status needs to be redrawn.
    - Calls [`status_pop_screen`](#status_pop_screen) to restore the previous screen state.
- **Output**:
    - The function does not return a value; it modifies the state of the client by freeing resources and updating flags.
- **Functions called**:
    - [`format_free`](format.c.driver.md#format_free)
    - [`status_pop_screen`](#status_pop_screen)


---
### status_prompt_update<!-- {{#callable:status_prompt_update}} -->
Updates the status prompt for a client with a new message and input.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose prompt is being updated.
    - `msg`: A string containing the new prompt message to be displayed.
    - `input`: A string representing the current input associated with the prompt.
- **Control Flow**:
    - Free the existing prompt string in the client structure.
    - Duplicate the new message string and assign it to the client's prompt string.
    - Free the existing prompt buffer in the client structure.
    - Expand the input string using the client's prompt formats and store it in a temporary variable.
    - Convert the temporary string to UTF-8 and assign it to the client's prompt buffer.
    - Calculate the length of the prompt buffer and store it in the client's prompt index.
    - Free the temporary string used for input expansion.
    - Reset the client's prompt history index to zero.
    - Set the client's flags to indicate that the status needs to be redrawn.
- **Output**:
    - The function does not return a value; it modifies the state of the client by updating its prompt string and buffer.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`format_expand_time`](format.c.driver.md#format_expand_time)
    - [`utf8_fromcstr`](utf8.c.driver.md#utf8_fromcstr)
    - [`utf8_strlen`](utf8.c.driver.md#utf8_strlen)


---
### status_prompt_redraw_character<!-- {{#callable:status_prompt_redraw_character}} -->
Redraws a character in the status prompt based on the current cursor position and character width.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `offset`: An unsigned integer representing the starting position offset for drawing the character.
    - `pwidth`: An unsigned integer indicating the total width of the prompt area.
    - `width`: A pointer to an unsigned integer that tracks the current width of the drawn characters.
    - `gc`: A pointer to a `grid_cell` structure that contains the graphical cell data for the character.
    - `ud`: A pointer to a `utf8_data` structure that contains the UTF-8 encoded character data to be drawn.
- **Control Flow**:
    - Check if the current width is less than the offset; if so, increment the width by the character's width and return 1.
    - If the current width is greater than or equal to the sum of the offset and prompt width, return 0.
    - Increment the current width by the character's width and check if it exceeds the prompt width; if so, return 0.
    - Extract the character from the `utf8_data` structure and check if it is a control character or delete character.
    - If it is a control character, set the `grid_cell` data to represent the control character visually.
    - If it is not a control character, copy the UTF-8 data to the `grid_cell` and write the cell to the screen.
- **Output**:
    - Returns 1 if the drawing can continue, or 0 if it cannot.
- **Functions called**:
    - [`utf8_copy`](utf8.c.driver.md#utf8_copy)
    - [`screen_write_cell`](screen-write.c.driver.md#screen_write_cell)


---
### status_prompt_redraw_quote<!-- {{#callable:status_prompt_redraw_quote}} -->
Redraws the quote indicator in the status prompt if necessary.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client context.
    - `pcursor`: An unsigned integer representing the position of the cursor in the prompt.
    - `ctx`: A pointer to the `screen_write_ctx` structure used for screen writing operations.
    - `offset`: An unsigned integer indicating the offset for drawing the prompt.
    - `pwidth`: An unsigned integer representing the width of the prompt area.
    - `width`: A pointer to an unsigned integer that will be updated with the current width of the drawn prompt.
    - `gc`: A pointer to a `grid_cell` structure that defines the graphical cell properties for drawing.
- **Control Flow**:
    - Checks if the `PROMPT_QUOTENEXT` flag is set in the client's prompt flags.
    - If the cursor's x-coordinate in the context matches the expected position (pcursor + 1), it prepares to draw the quote indicator.
    - Calls the [`status_prompt_redraw_character`](#status_prompt_redraw_character) function to handle the actual drawing of the quote character.
    - If the conditions are not met, it simply returns 1, indicating that redrawing can continue.
- **Output**:
    - Returns 1 if the drawing can continue, or 0 if it cannot.
- **Functions called**:
    - [`utf8_set`](utf8.c.driver.md#utf8_set)
    - [`status_prompt_redraw_character`](#status_prompt_redraw_character)


---
### status_prompt_redraw<!-- {{#callable:status_prompt_redraw}} -->
The `status_prompt_redraw` function updates and redraws the status line prompt for a client.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the client whose status prompt is to be redrawn.
- **Control Flow**:
    - Checks if the terminal size is valid (non-zero).
    - Copies the current active screen to `old_screen` for comparison after redrawing.
    - Determines the number of lines for the status line and initializes the screen.
    - Retrieves cursor color and style options from the session's options.
    - Calculates the prompt line position and applies the appropriate style based on the prompt mode.
    - Converts the prompt buffer to a string and expands the prompt string with the current input.
    - Draws the prompt on the screen, handling cursor positioning and character rendering.
    - Compares the new screen with the old screen to determine if a redraw is necessary.
- **Output**:
    - Returns 1 if the screen has changed and needs to be redrawn, or 0 if there are no changes.
- **Functions called**:
    - [`status_line_size`](#status_line_size)
    - [`screen_init`](screen.c.driver.md#screen_init)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`screen_set_cursor_style`](screen.c.driver.md#screen_set_cursor_style)
    - [`status_prompt_line_at`](#status_prompt_line_at)
    - [`style_apply`](style.c.driver.md#style_apply)
    - [`utf8_tocstr`](utf8.c.driver.md#utf8_tocstr)
    - [`format_expand_time`](format.c.driver.md#format_expand_time)
    - [`format_width`](format-draw.c.driver.md#format_width)
    - [`screen_write_start`](screen-write.c.driver.md#screen_write_start)
    - [`screen_write_fast_copy`](screen-write.c.driver.md#screen_write_fast_copy)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`screen_write_putc`](screen-write.c.driver.md#screen_write_putc)
    - [`format_draw`](format-draw.c.driver.md#format_draw)
    - [`utf8_strwidth`](utf8.c.driver.md#utf8_strwidth)
    - [`status_prompt_redraw_quote`](#status_prompt_redraw_quote)
    - [`status_prompt_redraw_character`](#status_prompt_redraw_character)
    - [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop)
    - [`grid_compare`](grid.c.driver.md#grid_compare)
    - [`screen_free`](screen.c.driver.md#screen_free)


---
### status_prompt_in_list<!-- {{#callable:status_prompt_in_list}} -->
The `status_prompt_in_list` function checks if a given character from a `utf8_data` structure is present in a specified string.
- **Inputs**:
    - `ws`: A pointer to a constant string where the function will search for the character.
    - `ud`: A pointer to a `utf8_data` structure containing the character to be checked.
- **Control Flow**:
    - The function first checks if the size and width of the character in `ud` are both equal to 1.
    - If either condition is false, the function returns 0, indicating the character is not valid for checking.
    - If the character is valid, it uses `strchr` to check if the character exists in the string `ws`.
    - The function returns 1 if the character is found in `ws`, otherwise it returns 0.
- **Output**:
    - The function returns an integer: 1 if the character is found in the string, and 0 if it is not or if the character is invalid.


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
Translates key inputs for a client in prompt mode, adapting them for either Emacs or Vi keybindings.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose key input is being processed.
    - `key`: The `key_code` representing the key input from the user.
    - `new_key`: A pointer to a `key_code` where the translated key will be stored.
- **Control Flow**:
    - Checks if the client is in `PROMPT_ENTRY` mode to handle key translations specific to that mode.
    - Uses a switch statement to match the input `key` against known key codes and translates them accordingly.
    - If the key is an escape sequence, it switches the prompt mode to `PROMPT_COMMAND` and sets a redraw flag.
    - If the key is not recognized, it defaults to returning the original key for further processing.
- **Output**:
    - Returns an integer indicating the result of the key translation: 0 to drop the key, 1 to process it as an Emacs key, or 2 to append it to the buffer.


---
### status_prompt_paste<!-- {{#callable:status_prompt_paste}} -->
The `status_prompt_paste` function pastes text from a clipboard-like buffer into a client's prompt buffer.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the client whose prompt buffer will be modified.
- **Control Flow**:
    - Calculate the current length of the prompt buffer using [`utf8_strlen`](utf8.c.driver.md#utf8_strlen).
    - Check if there is a saved prompt; if so, use it, otherwise retrieve the top paste buffer.
    - If the paste buffer is empty, return 0.
    - Iterate through the paste buffer data, converting it to UTF-8 and appending it to the client's prompt buffer.
    - If the prompt buffer is not empty, reallocate memory to accommodate the new data.
    - Insert the new data into the prompt buffer at the current index, adjusting the index accordingly.
    - Free any allocated memory for the UTF-8 data if it was not the saved prompt.
- **Output**:
    - Returns 1 on success, indicating that the paste operation was successful.
- **Functions called**:
    - [`utf8_strlen`](utf8.c.driver.md#utf8_strlen)
    - [`paste_get_top`](paste.c.driver.md#paste_get_top)
    - [`paste_buffer_data`](paste.c.driver.md#paste_buffer_data)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`utf8_open`](utf8.c.driver.md#utf8_open)
    - [`utf8_append`](utf8.c.driver.md#utf8_append)
    - [`utf8_set`](utf8.c.driver.md#utf8_set)


---
### status_prompt_replace_complete<!-- {{#callable:status_prompt_replace_complete}} -->
The `status_prompt_replace_complete` function replaces the current word in a prompt with a completed version based on the provided string.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose prompt is being modified.
    - `s`: A pointer to a string that represents the completion to be inserted; if NULL, the function attempts to complete the current word.
- **Control Flow**:
    - The function first determines the current cursor position in the prompt and identifies the word that the cursor is currently in.
    - It then checks if the provided string `s` is NULL; if it is, it extracts the current word from the prompt buffer.
    - If `s` is still NULL after attempting to extract the word, the function returns 0, indicating no completion was made.
    - If `s` is not NULL, it attempts to complete the word using the [`status_prompt_complete`](#status_prompt_complete) function.
    - The function then trims the identified word from the prompt buffer and reallocates the buffer to accommodate the new word.
    - Finally, it inserts the new word into the prompt buffer and updates the cursor position.
- **Output**:
    - The function returns 1 on successful completion of the word replacement, or 0 if no replacement was made.
- **Functions called**:
    - [`utf8_strlen`](utf8.c.driver.md#utf8_strlen)
    - [`status_prompt_space`](#status_prompt_space)
    - [`status_prompt_complete`](#status_prompt_complete)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`utf8_set`](utf8.c.driver.md#utf8_set)


---
### status_prompt_forward_word<!-- {{#callable:status_prompt_forward_word}} -->
The `status_prompt_forward_word` function moves the prompt index forward to the next word boundary in a command prompt.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the current client session.
    - `size_t size`: The total size of the prompt buffer.
    - `int vi`: An integer flag indicating whether the vi editing mode is active (1 for active, 0 for inactive).
    - `const char *separators`: A string containing characters that are considered as word separators.
- **Control Flow**:
    - The function initializes an index `idx` to the current prompt index and checks if the vi mode is inactive.
    - If vi mode is inactive, it skips any leading whitespace characters until it finds a non-whitespace character.
    - If the index reaches the end of the prompt buffer, it sets the prompt index to the end and returns.
    - It determines the character class of the current character at the prompt index to check if it is a separator or not.
    - The function then enters a loop to skip characters until it finds a space or a character that belongs to the opposite class.
    - If in vi mode, it continues to skip whitespace until it finds the start of the next word.
- **Output**:
    - The function updates the prompt index in the `client` structure to the new position after skipping to the next word boundary.
- **Functions called**:
    - [`status_prompt_space`](#status_prompt_space)
    - [`status_prompt_in_list`](#status_prompt_in_list)


---
### status_prompt_end_word<!-- {{#callable:status_prompt_end_word}} -->
The `status_prompt_end_word` function moves the prompt index to the end of the next word in the prompt buffer.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure that contains the prompt buffer and current prompt index.
    - `size_t size`: The total size of the prompt buffer.
    - `const char *separators`: A string containing characters that are considered as word separators.
- **Control Flow**:
    - The function first checks if the current prompt index is at the end of the buffer; if so, it returns immediately.
    - It then increments the index until it finds a non-space character, indicating the start of the next word.
    - Next, it determines if the character at the current index is a separator or not using the [`status_prompt_in_list`](#status_prompt_in_list) function.
    - The function continues to increment the index until it finds a space or a character that is of the opposite class (separator or non-separator).
    - Finally, it sets the prompt index to the last character of the identified word.
- **Output**:
    - The function does not return a value; it modifies the `prompt_index` field of the `client` structure to point to the end of the next word.
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
    - It then decrements `idx` until it finds a non-whitespace character, effectively locating the end of the current word.
    - Next, it checks if the character at the new `idx` is a separator using the [`status_prompt_in_list`](#status_prompt_in_list) function.
    - The function continues to decrement `idx` until it finds a space or a character that is of a different class (separator or non-separator) than the current character.
    - Finally, it updates the client's prompt index to the new `idx`, which is the start of the previous word.
- **Output**:
    - The function does not return a value; it modifies the `prompt_index` of the `client` structure to reflect the new position at the beginning of the previous word.
- **Functions called**:
    - [`status_prompt_space`](#status_prompt_space)
    - [`status_prompt_in_list`](#status_prompt_in_list)


---
### status_prompt_key<!-- {{#callable:status_prompt_key}} -->
The `status_prompt_key` function processes key inputs for a status prompt in a client session.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the current client session.
    - `key_code key`: The key code representing the key input from the user.
- **Control Flow**:
    - Checks if the prompt is in key mode and handles the key input accordingly.
    - Handles numeric input if the prompt is in numeric mode.
    - Processes various key commands such as navigation (left, right, home, end), deletion (backspace, delete), and control commands (copy, paste).
    - Handles special cases for different key modes (e.g., vi mode) and translates keys as necessary.
    - Updates the prompt buffer based on the key input, including appending characters and managing cursor position.
    - Calls the input callback function to handle the processed input and updates the status display.
- **Output**:
    - Returns 0 on success, indicating that the key was processed, or 1 if the input was handled and the prompt should be cleared.
- **Functions called**:
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)
    - [`status_prompt_clear`](#status_prompt_clear)
    - [`utf8_strlen`](utf8.c.driver.md#utf8_strlen)
    - [`utf8_tocstr`](utf8.c.driver.md#utf8_tocstr)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`status_prompt_translate_key`](#status_prompt_translate_key)
    - [`status_prompt_replace_complete`](#status_prompt_replace_complete)
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`status_prompt_space`](#status_prompt_space)
    - [`status_prompt_in_list`](#status_prompt_in_list)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`status_prompt_forward_word`](#status_prompt_forward_word)
    - [`status_prompt_end_word`](#status_prompt_end_word)
    - [`status_prompt_backward_word`](#status_prompt_backward_word)
    - [`status_prompt_up_history`](#status_prompt_up_history)
    - [`utf8_fromcstr`](utf8.c.driver.md#utf8_fromcstr)
    - [`status_prompt_down_history`](#status_prompt_down_history)
    - [`status_prompt_paste`](#status_prompt_paste)
    - [`utf8_copy`](utf8.c.driver.md#utf8_copy)
    - [`status_prompt_add_history`](#status_prompt_add_history)
    - [`utf8_set`](utf8.c.driver.md#utf8_set)
    - [`utf8_to_data`](utf8.c.driver.md#utf8_to_data)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### status_prompt_up_history<!-- {{#callable:status_prompt_up_history}} -->
Retrieves the previous entry from the status prompt history.
- **Inputs**:
    - `idx`: A pointer to an array of unsigned integers representing the current index for each prompt type.
    - `type`: An unsigned integer representing the type of prompt whose history is being accessed.
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
    - `type`: An unsigned integer representing the specific type of prompt for which the history is being accessed.
- **Control Flow**:
    - Checks if the history size for the specified type is zero or if the current index is zero, returning an empty string if true.
    - Decrements the index for the specified type.
    - Checks if the decremented index is zero, returning an empty string if true.
    - Returns the history entry corresponding to the decremented index.
- **Output**:
    - Returns a pointer to the previous history entry as a string, or an empty string if there are no entries.


---
### status_prompt_add_history<!-- {{#callable:status_prompt_add_history}} -->
The `status_prompt_add_history` function manages the addition of a new line to the command prompt history.
- **Inputs**:
    - `line`: A pointer to a string representing the line to be added to the history.
    - `type`: An unsigned integer representing the type of prompt for which the history is being maintained.
- **Control Flow**:
    - The function first checks if the new line is identical to the last line in the history for the given type; if so, it sets a flag to avoid adding a duplicate.
    - It retrieves the maximum history limit from global options and determines the new size of the history list based on whether the limit has been reached.
    - If the new line is to be added and the history size exceeds the limit, it frees the oldest entries in the history.
    - The function reallocates memory for the history list if the size has changed and adds the new line to the end of the list if it is not a duplicate.
- **Output**:
    - The function does not return a value; it modifies the global history list and size for the specified prompt type.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### status_prompt_add_list<!-- {{#callable:status_prompt_add_list}} -->
The `status_prompt_add_list` function adds a unique string to a dynamically allocated list.
- **Inputs**:
    - `list`: A pointer to a pointer to a dynamically allocated array of strings where the new string will be added.
    - `size`: A pointer to an unsigned integer representing the current size of the list.
    - `s`: A constant string that is to be added to the list if it is not already present.
- **Control Flow**:
    - The function iterates through the existing list to check if the string `s` is already present.
    - If `s` is found in the list, the function returns immediately without making any changes.
    - If `s` is not found, the function reallocates the list to accommodate one more string.
    - The new string `s` is duplicated and added to the end of the list, and the size is incremented.
- **Output**:
    - The function does not return a value; it modifies the list and size in place.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### status_prompt_complete_list<!-- {{#callable:status_prompt_complete_list}} -->
The `status_prompt_complete_list` function generates a list of command completions based on a given input string.
- **Inputs**:
    - `size`: A pointer to a `u_int` variable that will hold the size of the completion list.
    - `s`: A constant string representing the input for which completions are being generated.
    - `at_start`: An integer flag indicating whether to return completions that start with the input string.
- **Control Flow**:
    - The function initializes the size of the completion list to zero.
    - It iterates over a command table, checking if each command name or alias starts with the input string, adding matches to the completion list.
    - It retrieves command aliases from global options and checks if they match the input string, adding them to the list if they do.
    - If `at_start` is false, it further checks options table entries and predefined layout strings for matches, adding them to the list.
    - Finally, it returns the populated completion list.
- **Output**:
    - The function returns a dynamically allocated array of strings containing the matched completions.
- **Functions called**:
    - [`status_prompt_add_list`](#status_prompt_add_list)
    - [`options_get_only`](options.c.driver.md#options_get_only)
    - [`options_array_first`](options.c.driver.md#options_array_first)
    - [`options_array_item_value`](options.c.driver.md#options_array_item_value)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`options_array_next`](options.c.driver.md#options_array_next)


---
### status_prompt_complete_prefix<!-- {{#callable:status_prompt_complete_prefix}} -->
The `status_prompt_complete_prefix` function finds the longest common prefix among a list of strings.
- **Inputs**:
    - `list`: An array of strings (char**) representing the list of strings to compare.
    - `size`: An unsigned integer (u_int) representing the number of strings in the list.
- **Control Flow**:
    - Checks if the input `list` is NULL or if `size` is 0, returning NULL if true.
    - Duplicates the first string in the list to `out`.
    - Iterates through the remaining strings in the list starting from the second string.
    - For each string, it determines the length to compare based on the shorter of the current `out` and the string being compared.
    - Compares characters from the end of the strings towards the beginning, truncating `out` when characters do not match.
- **Output**:
    - Returns a newly allocated string containing the longest common prefix of the input strings, or NULL if no common prefix exists.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### status_prompt_menu_callback<!-- {{#callable:status_prompt_menu_callback}} -->
The `status_prompt_menu_callback` function handles user input from a menu for status prompt completion.
- **Inputs**:
    - `menu`: A pointer to a `menu` structure, which represents the menu being interacted with.
    - `idx`: An unsigned integer representing the index of the selected item in the menu.
    - `key`: A `key_code` representing the key pressed by the user.
    - `data`: A pointer to user-defined data, which is expected to be a pointer to a `status_prompt_menu` structure.
- **Control Flow**:
    - The function first checks if the `key` is not `KEYC_NONE`, indicating that a key was pressed.
    - If a key was pressed, it adjusts the `idx` by adding the `start` value from the `status_prompt_menu` structure.
    - It then creates a string `s` based on the selected menu item, optionally prepending a flag character if it exists.
    - If the prompt type of the client is `PROMPT_TYPE_WINDOW_TARGET`, it updates the client's prompt buffer with the new string and sets the redraw flag.
    - If the prompt type is not `PROMPT_TYPE_WINDOW_TARGET`, it attempts to replace the current prompt with the new string using [`status_prompt_replace_complete`](#status_prompt_replace_complete).
    - Finally, it frees the allocated memory for the menu items and the list.
- **Output**:
    - The function does not return a value; it modifies the state of the client and the prompt based on user input.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`utf8_fromcstr`](utf8.c.driver.md#utf8_fromcstr)
    - [`utf8_strlen`](utf8.c.driver.md#utf8_strlen)
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
    - Checks if the available screen space is insufficient to display the menu; if so, returns 0.
    - Allocates memory for a `status_prompt_menu` structure and initializes it with the provided parameters.
    - Calculates the height of the menu based on available screen space and the number of items.
    - Creates a new menu and populates it with items from the list, assigning keys to each item.
    - Determines the vertical position for displaying the menu based on user preferences.
    - Adjusts the horizontal offset based on the width of the prompt string.
    - Displays the menu using the [`menu_display`](menu.c.driver.md#menu_display) function, passing the necessary parameters and callback.
    - If the menu display fails, frees allocated resources and returns 0; otherwise, returns 1 to indicate success.
- **Output**:
    - Returns 1 if the menu was successfully displayed, or 0 if it was not.
- **Functions called**:
    - [`status_line_size`](#status_line_size)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`menu_create`](menu.c.driver.md#menu_create)
    - [`menu_add_item`](menu.c.driver.md#menu_add_item)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`utf8_cstrwidth`](utf8.c.driver.md#utf8_cstrwidth)
    - [`menu_display`](menu.c.driver.md#menu_display)
    - [`menu_free`](menu.c.driver.md#menu_free)


---
### status_prompt_complete_window_menu<!-- {{#callable:status_prompt_complete_window_menu}} -->
The `status_prompt_complete_window_menu` function displays a menu for completing window names based on user input.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the current client.
    - `struct session *s`: A pointer to the `session` structure containing the session information.
    - `const char *word`: A string representing the current input word for which completion is requested.
    - `u_int offset`: An unsigned integer representing the offset for displaying the menu.
    - `char flag`: A character flag that may modify the behavior of the completion.
- **Control Flow**:
    - Checks if the terminal height is sufficient to display the menu; if not, returns NULL.
    - Allocates memory for a `status_prompt_menu` structure and initializes it with the client and flag.
    - Calculates the maximum height of the menu based on the terminal size and the number of status lines.
    - Iterates over the window links in the session, filtering them based on the input word if provided.
    - For each matching window, constructs a display string and adds it to the menu items.
    - If no items are found, frees allocated resources and returns NULL.
    - If only one item is found, returns that item directly, potentially modifying it with the flag.
    - Displays the menu using the [`menu_display`](menu.c.driver.md#menu_display) function, handling the layout and position based on terminal size.
- **Output**:
    - Returns a string representing the selected window name if a single match is found, or NULL if no matches are available.
- **Functions called**:
    - [`status_line_size`](#status_line_size)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`menu_create`](menu.c.driver.md#menu_create)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`menu_add_item`](menu.c.driver.md#menu_add_item)
    - [`menu_free`](menu.c.driver.md#menu_free)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`utf8_cstrwidth`](utf8.c.driver.md#utf8_cstrwidth)
    - [`menu_display`](menu.c.driver.md#menu_display)


---
### status_prompt_complete_sort<!-- {{#callable:status_prompt_complete_sort}} -->
The `status_prompt_complete_sort` function compares two strings for sorting purposes.
- **Inputs**:
    - `a`: A pointer to the first string to compare.
    - `b`: A pointer to the second string to compare.
- **Control Flow**:
    - The function casts the input pointers to `const char **` to access the strings they point to.
    - It uses the `strcmp` function to compare the two strings pointed to by `aa` and `bb`.
    - The result of `strcmp` is returned, which indicates the lexicographical order of the strings.
- **Output**:
    - The function returns an integer: a negative value if the first string is less than the second, zero if they are equal, and a positive value if the first string is greater than the second.


---
### status_prompt_complete_session<!-- {{#callable:status_prompt_complete_session}} -->
Completes session names or IDs based on a given string.
- **Inputs**:
    - `list`: A pointer to a pointer of character arrays where the completion results will be stored.
    - `size`: A pointer to an unsigned integer that keeps track of the number of completions found.
    - `s`: A constant character pointer representing the input string to match against session names or IDs.
    - `flag`: A character that may be used to modify the output format of the completion.
- **Control Flow**:
    - Iterates over all sessions using `RB_FOREACH` to check each session's name against the input string.
    - If the input string is empty or matches the beginning of a session name, it adds the session name to the completion list.
    - If the input string starts with '$', it checks if the subsequent characters match the session ID and adds it to the list if they do.
    - After populating the list, it calls [`status_prompt_complete_prefix`](#status_prompt_complete_prefix) to find the common prefix of the completions.
    - If a flag is provided, it formats the output by prepending the flag to the common prefix.
- **Output**:
    - Returns a dynamically allocated string containing the common prefix of the completed session names or IDs, or NULL if no completions were found.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
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
    - If the prompt type is not target or window target and the word does not start with '-t' or '-s', generate a completion list using [`status_prompt_complete_list`](#status_prompt_complete_list).
    - If the prompt type is target or window target, adjust the input string and flag accordingly.
    - If the prompt type is window target, call [`status_prompt_complete_window_menu`](#status_prompt_complete_window_menu) to display a menu for window completion.
    - If the input contains a colon, determine if it is a session completion or a window completion based on the presence of a period after the colon.
    - If a session is found, call [`status_prompt_complete_window_menu`](#status_prompt_complete_window_menu) to show the corresponding window menu.
    - Sort the completion list if it is not empty and log the results.
    - If the generated output matches the input word, free the output and set it to NULL.
    - If there is an output or if the completion list menu is not displayed, free the list and return the output.
- **Output**:
    - Returns a dynamically allocated string containing the completed word or NULL if no completion is possible.
- **Functions called**:
    - [`status_prompt_complete_list`](#status_prompt_complete_list)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`status_prompt_complete_prefix`](#status_prompt_complete_prefix)
    - [`status_prompt_complete_window_menu`](#status_prompt_complete_window_menu)
    - [`status_prompt_complete_session`](#status_prompt_complete_session)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`session_find`](session.c.driver.md#session_find)
    - [`status_prompt_complete_list_menu`](#status_prompt_complete_list_menu)


---
### status_prompt_type<!-- {{#callable:status_prompt_type}} -->
The `status_prompt_type` function maps a string representation of a prompt type to its corresponding enum value.
- **Inputs**:
    - `type`: A constant character pointer representing the string name of the prompt type to be matched.
- **Control Flow**:
    - Iterates through all defined prompt types using a loop indexed by `i`.
    - Compares the input string `type` with the string representation of each prompt type using `strcmp`.
    - If a match is found, returns the corresponding enum value `i`.
    - If no match is found after checking all types, returns `PROMPT_TYPE_INVALID`.
- **Output**:
    - Returns an enum value of type `prompt_type` corresponding to the input string, or `PROMPT_TYPE_INVALID` if no match is found.
- **Functions called**:
    - [`status_prompt_type_string`](#status_prompt_type_string)


---
### status_prompt_type_string<!-- {{#callable:status_prompt_type_string}} -->
Returns the string representation of a prompt type based on its index.
- **Inputs**:
    - `type`: An unsigned integer representing the index of the prompt type.
- **Control Flow**:
    - Checks if the input `type` is greater than or equal to `PROMPT_NTYPES`.
    - If the condition is true, returns the string 'invalid'.
    - If the condition is false, returns the corresponding string from the `prompt_type_strings` array.
- **Output**:
    - A constant string representing the prompt type, or 'invalid' if the input index is out of bounds.


