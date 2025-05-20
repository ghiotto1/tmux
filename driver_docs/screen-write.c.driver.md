# Purpose
This C source code file is part of the `tmux` project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file provides a comprehensive set of functions for handling screen writing operations, which are essential for rendering text and managing the display within a `tmux` session. The code includes functions for manipulating the screen buffer, such as inserting and deleting characters and lines, scrolling, and managing cursor movements. It also handles more complex operations like drawing boxes, menus, and handling UTF-8 character encoding, which is crucial for supporting a wide range of characters and symbols.

The file defines several static functions and structures that are used internally to manage screen writing contexts and operations. It includes functions for initializing and finalizing screen writing sessions, managing the cursor position, and handling text attributes like colors and character sets. The code also provides mechanisms for handling external clipboard operations and raw string writing, which are important for integrating with other system components and ensuring smooth user interactions. Overall, this file is a critical component of the `tmux` codebase, providing the necessary functionality to render and manage terminal content efficiently.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### screen_write_collect_trim
- **Type**: `static struct screen_write_citem *`
- **Description**: The `screen_write_collect_trim` function is a static function that returns a pointer to a `screen_write_citem` structure. It is used to manage and manipulate collected screen write items, specifically trimming them based on certain conditions.
- **Use**: This function is used internally to trim collected screen write items within a specified range.


---
### screen_write_citem_freelist
- **Type**: `TAILQ_HEAD(, screen_write_citem)`
- **Description**: The `screen_write_citem_freelist` is a global variable that represents a tail queue head for managing a list of `screen_write_citem` structures. It is initialized using the `TAILQ_HEAD_INITIALIZER` macro, which sets up the queue to be empty initially.
- **Use**: This variable is used to manage a free list of `screen_write_citem` structures, allowing for efficient allocation and deallocation of these items within the program.


# Data Structures

---
### screen_write_citem
- **Type**: `struct`
- **Members**:
    - `x`: The x-coordinate of the item on the screen.
    - `wrapped`: Indicates if the line is wrapped.
    - `type`: Specifies the type of the item, either TEXT or CLEAR.
    - `used`: The number of units used by this item.
    - `bg`: The background color of the item.
    - `gc`: A grid_cell structure representing the cell's attributes.
    - `entry`: A TAILQ_ENTRY for linking items in a queue.
- **Description**: The `screen_write_citem` structure is used to represent an item in a screen writing context, typically within a terminal or text-based user interface. It holds information about the position, type, and visual attributes of a screen item, including its x-coordinate, whether it is wrapped, its type (TEXT or CLEAR), the number of units it occupies, and its background color. Additionally, it contains a `grid_cell` structure for detailed cell attributes and a `TAILQ_ENTRY` for managing the item in a linked list queue.


---
### screen_write_cline
- **Type**: `struct`
- **Members**:
    - `data`: A pointer to a character array, likely used to store line data.
    - `items`: A tail queue head for managing a list of `screen_write_citem` structures.
- **Description**: The `screen_write_cline` structure is designed to represent a line of text in a screen writing context, with `data` holding the actual text content and `items` managing a list of `screen_write_citem` structures that describe individual items or segments within the line. This structure is part of a larger system for managing screen output, likely in a terminal or text-based user interface, where lines of text need to be dynamically managed and manipulated.


# Functions

---
### screen_write_get_citem<!-- {{#callable:screen_write_get_citem}} -->
The `screen_write_get_citem` function retrieves a `screen_write_citem` structure from a free list or allocates a new one if the list is empty.
- **Inputs**:
    - None
- **Control Flow**:
    - The function first attempts to get the first item from the `screen_write_citem_freelist` using `TAILQ_FIRST`.
    - If an item is found (i.e., `ci` is not NULL), it is removed from the free list using `TAILQ_REMOVE`, and its memory is cleared with `memset`.
    - The cleared item is then returned.
    - If no item is found in the free list, a new `screen_write_citem` is allocated using [`xcalloc`](xmalloc.c.driver.md#xcalloc) and returned.
- **Output**:
    - The function returns a pointer to a `screen_write_citem`, either from the free list or a newly allocated one.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### screen_write_free_citem<!-- {{#callable:screen_write_free_citem}} -->
The `screen_write_free_citem` function adds a `screen_write_citem` structure to the end of a free list.
- **Inputs**:
    - `ci`: A pointer to a `screen_write_citem` structure that is to be added to the free list.
- **Control Flow**:
    - The function directly calls `TAILQ_INSERT_TAIL` to insert the `ci` item at the end of the `screen_write_citem_freelist` queue.
    - No conditional logic or loops are present in this function.
- **Output**:
    - The function does not return a value; it modifies the state of the free list by adding the provided `screen_write_citem` to it.


---
### screen_write_offset_timer<!-- {{#callable:screen_write_offset_timer}} -->
The `screen_write_offset_timer` function updates the window offset for a given window.
- **Inputs**:
    - `fd`: An integer file descriptor, which is unused in this function.
    - `events`: A short integer representing event types, which is also unused.
    - `data`: A pointer to a `void` type that is expected to point to a `window` structure.
- **Control Flow**:
    - The function retrieves the `window` structure from the `data` pointer.
    - It then calls the [`tty_update_window_offset`](tty.c.driver.md#tty_update_window_offset) function, passing the retrieved `window` as an argument.
- **Output**:
    - The function does not return a value; it performs an update operation on the window offset.
- **Functions called**:
    - [`tty_update_window_offset`](tty.c.driver.md#tty_update_window_offset)


---
### screen_write_set_cursor<!-- {{#callable:screen_write_set_cursor}} -->
Sets the cursor position on the screen.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `cx`: An integer representing the x-coordinate (column) of the cursor position.
    - `cy`: An integer representing the y-coordinate (row) of the cursor position.
- **Control Flow**:
    - Checks if the new cursor position is the same as the current position; if so, it returns early.
    - If the x-coordinate (cx) is valid (not -1), it checks if it exceeds the screen width and adjusts it if necessary.
    - If the y-coordinate (cy) is valid (not -1), it checks if it exceeds the screen height and adjusts it if necessary.
    - If the `window_pane` pointer in the context is NULL, it returns early.
    - If the `offset_timer` for the window is not initialized, it sets the timer to call `screen_write_offset_timer`.
    - If the timer is not already pending, it adds the timer with a delay of 10 milliseconds.
- **Output**:
    - The function does not return a value; it modifies the cursor position in the screen context and manages the timer for screen updates.


---
### screen_write_redraw_cb<!-- {{#callable:screen_write_redraw_cb}} -->
The `screen_write_redraw_cb` function marks a window pane for redrawing.
- **Inputs**:
    - `ttyctx`: A pointer to a `tty_ctx` structure that contains context information, including a pointer to the associated `window_pane`.
- **Control Flow**:
    - The function retrieves the `window_pane` from the `tty_ctx` structure using the `arg` field.
    - If the `window_pane` is not NULL, it sets the `PANE_REDRAW` flag in the `flags` member of the `window_pane`.
- **Output**:
    - The function does not return a value; it modifies the state of the `window_pane` by setting a flag to indicate that it needs to be redrawn.


---
### screen_write_set_client_cb<!-- {{#callable:screen_write_set_client_cb}} -->
Sets the client callback for screen writing based on the current TTY context and client.
- **Inputs**:
    - `ttyctx`: A pointer to a `tty_ctx` structure that contains the context for the terminal.
    - `c`: A pointer to a `client` structure representing the client associated with the TTY.
- **Control Flow**:
    - Checks if invisible panes are allowed; if so, verifies if the client's session has the window pane.
    - If the current window of the client does not match the window pane, returns 0.
    - Checks if the layout cell of the window pane is NULL; if so, returns 0.
    - Checks if the window pane needs to be redrawn or dropped; if so, returns -1.
    - If the client has a redraw flag set, marks the pane for redrawing and returns -1.
    - Calculates the window offset and updates the TTY context offsets based on the window pane's offsets.
    - If the status line is not at line 0, adjusts the Y offset in the TTY context.
- **Output**:
    - Returns 1 if the client callback is successfully set, 0 if the client is not associated with the window pane, or -1 if the pane needs to be redrawn or dropped.
- **Functions called**:
    - [`session_has`](session.c.driver.md#session_has)
    - [`tty_window_offset`](tty.c.driver.md#tty_window_offset)
    - [`status_at_line`](status.c.driver.md#status_at_line)
    - [`status_line_size`](status.c.driver.md#status_line_size)


---
### screen_write_initctx<!-- {{#callable:screen_write_initctx}} -->
Initializes the `tty_ctx` structure for screen writing.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `ttyctx`: A pointer to the `tty_ctx` structure that will be initialized with screen dimensions and other properties.
    - `sync`: An integer flag indicating whether to synchronize updates.
- **Control Flow**:
    - Clears the `ttyctx` structure by setting its memory to zero.
    - Assigns the screen pointer from `ctx` to `ttyctx` and retrieves the screen dimensions.
    - Copies default cell properties from `grid_default_cell` to `ttyctx`.
    - If `ctx->init_ctx_cb` is not NULL, it calls the callback function with `ctx` and `ttyctx` as arguments.
    - If a palette is defined in `ttyctx`, it updates the foreground and background colors if they are set to default values.
    - If `ctx->init_ctx_cb` is NULL, it sets default callbacks for redrawing and client setting.
    - If the `SCREEN_WRITE_SYNC` flag is not set in `ctx->flags`, it determines the synchronization behavior based on the active window pane and sets the synchronization command accordingly.
- **Output**:
    - The function does not return a value but initializes the `ttyctx` structure with the current screen context and prepares it for subsequent screen writing operations.
- **Functions called**:
    - [`tty_default_colours`](tty.c.driver.md#tty_default_colours)
    - [`tty_write`](tty.c.driver.md#tty_write)


---
### screen_write_make_list<!-- {{#callable:screen_write_make_list}} -->
The `screen_write_make_list` function initializes a write list for a given screen structure.
- **Inputs**:
    - `s`: A pointer to a `struct screen` that represents the screen for which the write list is being created.
- **Control Flow**:
    - Allocates memory for the `write_list` array using [`xcalloc`](xmalloc.c.driver.md#xcalloc), with the size determined by the height of the screen.
    - Iterates over each row of the screen (from 0 to the height of the screen) and initializes a doubly linked list (`TAILQ_INIT`) for each entry in the `write_list`.
- **Output**:
    - The function does not return a value; it modifies the `write_list` field of the `screen` structure to prepare it for subsequent writing operations.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### screen_write_free_list<!-- {{#callable:screen_write_free_list}} -->
The `screen_write_free_list` function frees the memory allocated for the write list of a `screen` structure.
- **Inputs**:
    - `s`: A pointer to a `screen` structure whose write list is to be freed.
- **Control Flow**:
    - Iterates over each row of the screen's write list using a loop that runs from 0 to the height of the screen.
    - For each row, it frees the memory allocated for the `data` field of the `write_list` entry.
    - After freeing all individual row data, it frees the entire `write_list` array.
- **Output**:
    - This function does not return a value; it performs memory deallocation to prevent memory leaks.


---
### screen_write_init<!-- {{#callable:screen_write_init}} -->
Initializes the `screen_write_ctx` structure for screen writing.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `s`: A pointer to the `screen` structure that represents the screen to be written to.
- **Control Flow**:
    - Clears the `ctx` structure by setting its memory to zero using `memset`.
    - Assigns the provided `screen` pointer `s` to the `ctx` structure.
    - Checks if the `write_list` of the `screen` is NULL; if so, it calls [`screen_write_make_list`](#screen_write_make_list) to create it.
    - Retrieves a new `screen_write_citem` using [`screen_write_get_citem`](#screen_write_get_citem) and assigns it to `ctx->item`.
    - Initializes `ctx->scrolled` to 0 and sets `ctx->bg` to 8.
- **Output**:
    - The function does not return a value; it initializes the context for screen writing operations.
- **Functions called**:
    - [`screen_write_make_list`](#screen_write_make_list)
    - [`screen_write_get_citem`](#screen_write_get_citem)


---
### screen_write_start_pane<!-- {{#callable:screen_write_start_pane}} -->
Initializes the screen writing context for a specific pane.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `wp`: A pointer to the `window_pane` structure representing the pane to be written to.
    - `s`: A pointer to the `screen` structure that represents the screen to be used; if NULL, the screen from the pane is used.
- **Control Flow**:
    - If the provided screen pointer `s` is NULL, it is set to the screen associated with the given pane `wp`.
    - The [`screen_write_init`](#screen_write_init) function is called to initialize the writing context with the determined screen.
    - The `wp` field of the context `ctx` is set to the provided pane `wp`.
    - If the logging level is not zero, a debug log is generated that includes the size of the screen and the position of the pane.
- **Output**:
    - The function does not return a value; it modifies the `screen_write_ctx` structure to prepare it for writing operations.
- **Functions called**:
    - [`screen_write_init`](#screen_write_init)
    - [`log_get_level`](log.c.driver.md#log_get_level)


---
### screen_write_start_callback<!-- {{#callable:screen_write_start_callback}} -->
Initializes the screen writing context with a callback function.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `s`: A pointer to the `screen` structure representing the screen to be written to.
    - `cb`: A callback function of type `screen_write_init_ctx_cb` that will be called during initialization.
    - `arg`: A pointer to user-defined data that will be passed to the callback function.
- **Control Flow**:
    - Calls [`screen_write_init`](#screen_write_init) to initialize the context with the provided screen.
    - Sets the `init_ctx_cb` member of the context to the provided callback function.
    - Sets the `arg` member of the context to the provided argument.
    - If the logging level is greater than 0, logs the size of the screen along with the callback information.
- **Output**:
    - The function does not return a value; it modifies the `screen_write_ctx` structure and may log debug information.
- **Functions called**:
    - [`screen_write_init`](#screen_write_init)
    - [`log_get_level`](log.c.driver.md#log_get_level)


---
### screen_write_start<!-- {{#callable:screen_write_start}} -->
Initializes the screen writing context for output.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `s`: A pointer to a `screen` structure that represents the screen to be written to.
- **Control Flow**:
    - Calls [`screen_write_init`](#screen_write_init) to initialize the writing context with the provided screen.
    - Checks the logging level; if it is not zero, logs the size of the screen being initialized.
- **Output**:
    - The function does not return a value; it initializes the context for screen writing and may log debug information.
- **Functions called**:
    - [`screen_write_init`](#screen_write_init)
    - [`log_get_level`](log.c.driver.md#log_get_level)


---
### screen_write_stop<!-- {{#callable:screen_write_stop}} -->
The `screen_write_stop` function finalizes the screen writing process by collecting and flushing any remaining output and freeing associated resources.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing operations.
- **Control Flow**:
    - Calls `screen_write_collect_end(ctx)` to finalize the collection of screen write items.
    - Calls `screen_write_collect_flush(ctx, 0, __func__)` to flush any collected items to the screen.
    - Calls `screen_write_free_citem(ctx->item)` to free the current collected item.
- **Output**:
    - The function does not return a value; it performs operations that affect the screen output and resource management.
- **Functions called**:
    - [`screen_write_collect_end`](#screen_write_collect_end)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_write_free_citem`](#screen_write_free_citem)


---
### screen_write_reset<!-- {{#callable:screen_write_reset}} -->
Resets the screen state and cursor position for the given context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
- **Control Flow**:
    - Calls [`screen_reset_tabs`](screen.c.driver.md#screen_reset_tabs) to reset tab stops for the screen.
    - Sets the scroll region for the screen to cover the entire height using [`screen_write_scrollregion`](#screen_write_scrollregion).
    - Initializes the screen mode to include cursor visibility and line wrapping.
    - Checks if the 'extended-keys' option is set to 2 and modifies the screen mode accordingly.
    - Clears the screen using [`screen_write_clearscreen`](#screen_write_clearscreen).
    - Sets the cursor position to the top-left corner of the screen using [`screen_write_set_cursor`](#screen_write_set_cursor).
- **Output**:
    - The function does not return a value; it modifies the state of the screen and cursor based on the provided context.
- **Functions called**:
    - [`screen_reset_tabs`](screen.c.driver.md#screen_reset_tabs)
    - [`screen_write_scrollregion`](#screen_write_scrollregion)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`screen_write_clearscreen`](#screen_write_clearscreen)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_putc<!-- {{#callable:screen_write_putc}} -->
The `screen_write_putc` function writes a single character to the screen at the current cursor position.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `gcp`: A pointer to a `grid_cell` structure that contains the attributes and data for the cell to be written.
    - `ch`: The character (of type `u_char`) to be written to the screen.
- **Control Flow**:
    - A local variable `gc` of type `grid_cell` is declared to hold a copy of the input cell attributes.
    - The `memcpy` function is used to copy the contents of `gcp` into `gc`.
    - The [`utf8_set`](utf8.c.driver.md#utf8_set) function is called to set the character data of `gc` to the provided character `ch`.
    - The [`screen_write_cell`](#screen_write_cell) function is called to write the modified `gc` to the screen.
- **Output**:
    - The function does not return a value; it directly modifies the screen by writing the specified character with the given attributes at the current cursor position.
- **Functions called**:
    - [`utf8_set`](utf8.c.driver.md#utf8_set)
    - [`screen_write_cell`](#screen_write_cell)


---
### screen_write_strlen<!-- {{#callable:screen_write_strlen}} -->
Calculates the display length of a formatted string considering UTF-8 encoding.
- **Inputs**:
    - `fmt`: A format string that specifies how to format the subsequent arguments.
    - `ap`: A variable argument list that contains the values to be formatted according to the format string.
- **Control Flow**:
    - Initializes a variable argument list using `va_start` and formats the string into `msg` using [`xvasprintf`](xmalloc.c.driver.md#xvasprintf).
    - Iterates through each character in the formatted string `msg` until the null terminator is reached.
    - Checks if the character is a UTF-8 character (greater than 0x7F) and processes it accordingly using [`utf8_open`](utf8.c.driver.md#utf8_open) and [`utf8_append`](utf8.c.driver.md#utf8_append).
    - If the character is a standard ASCII character (0x20 to 0x7E), increments the size count for display length.
    - Handles tab characters separately by incrementing the size count.
    - Finally, frees the allocated memory for `msg` and returns the calculated size.
- **Output**:
    - Returns the total display length of the formatted string, accounting for UTF-8 character widths.
- **Functions called**:
    - [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)
    - [`utf8_open`](utf8.c.driver.md#utf8_open)
    - [`utf8_append`](utf8.c.driver.md#utf8_append)


---
### screen_write_text<!-- {{#callable:screen_write_text}} -->
The `screen_write_text` function writes formatted text to a specified area of the screen, handling line wrapping and cursor movement.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `cx`: The x-coordinate (column) on the screen where the text should start being written.
    - `width`: The maximum width (in characters) of the area where text can be written.
    - `lines`: The number of lines available for writing text.
    - `more`: An integer flag indicating if more text will follow after this call.
    - `gcp`: A pointer to a `grid_cell` structure that defines the attributes (like color) of the text being written.
    - `fmt`: A format string that specifies how to format the text to be written.
- **Control Flow**:
    - The function starts by initializing a `va_list` to handle variable arguments and formats the input string using [`xvasprintf`](xmalloc.c.driver.md#xvasprintf).
    - It converts the formatted string into UTF-8 data using [`utf8_fromcstr`](utf8.c.driver.md#utf8_fromcstr).
    - A loop is initiated to write text line by line until the specified number of lines is reached or the text is fully consumed.
    - Within the loop, it calculates how much text can fit on the current line, checking for line breaks and spaces.
    - If the end of the text is reached or the maximum number of lines is written, the loop breaks.
    - After writing, it checks if there is more text to come and adjusts the cursor position accordingly.
- **Output**:
    - The function returns 1 if text was successfully written and more text is expected, or 0 if it reaches the end of the specified lines or if the text was not fully consumed.
- **Functions called**:
    - [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)
    - [`utf8_fromcstr`](utf8.c.driver.md#utf8_fromcstr)
    - [`utf8_copy`](utf8.c.driver.md#utf8_copy)
    - [`screen_write_cell`](#screen_write_cell)
    - [`screen_write_cursormove`](#screen_write_cursormove)


---
### screen_write_puts<!-- {{#callable:screen_write_puts}} -->
The `screen_write_puts` function writes a formatted string to the screen using a specified grid cell context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `gcp`: A pointer to a `grid_cell` structure that defines the attributes of the cell to be used for writing.
    - `fmt`: A format string that specifies how subsequent arguments are converted for output.
- **Control Flow**:
    - The function starts by initializing a variable argument list using `va_start` with the format string `fmt`.
    - It then calls the [`screen_write_vnputs`](#screen_write_vnputs) function, passing the context, a maximum length of -1 (indicating no limit), the grid cell pointer, the format string, and the argument list.
    - Finally, it ends the variable argument list with `va_end`.
- **Output**:
    - The function does not return a value; it writes the formatted string to the screen based on the provided context and grid cell attributes.
- **Functions called**:
    - [`screen_write_vnputs`](#screen_write_vnputs)


---
### screen_write_nputs<!-- {{#callable:screen_write_nputs}} -->
The `screen_write_nputs` function writes a formatted string to the screen with an optional maximum length.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `maxlen`: A `ssize_t` value that specifies the maximum number of characters to write; if -1, there is no limit.
    - `gcp`: A pointer to a `grid_cell` structure that defines the attributes of the characters to be written.
    - `fmt`: A format string that specifies how subsequent arguments are converted for output.
    - `...`: A variable number of additional arguments that are formatted according to the `fmt` string.
- **Control Flow**:
    - The function begins by initializing a variable argument list using `va_start` with the provided format string `fmt`.
    - It then calls the [`screen_write_vnputs`](#screen_write_vnputs) function, passing the context, maximum length, grid cell, format string, and the argument list.
    - Finally, it cleans up the variable argument list using `va_end`.
- **Output**:
    - The function does not return a value; it writes formatted output directly to the screen based on the provided context and parameters.
- **Functions called**:
    - [`screen_write_vnputs`](#screen_write_vnputs)


---
### screen_write_vnputs<!-- {{#callable:screen_write_vnputs}} -->
The `screen_write_vnputs` function writes a formatted string to a screen context, handling UTF-8 characters and respecting a maximum length.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `maxlen`: A `ssize_t` value that specifies the maximum number of characters to write; -1 indicates no limit.
    - `gcp`: A pointer to a `grid_cell` structure that defines the attributes of the characters to be written.
    - `fmt`: A format string that specifies how to format the output.
    - `ap`: A `va_list` containing the arguments to be formatted according to `fmt`.
- **Control Flow**:
    - The function begins by copying the attributes from the provided `grid_cell` into a local variable.
    - It then formats the input string using [`xvasprintf`](xmalloc.c.driver.md#xvasprintf), storing the result in `msg`.
    - A loop iterates over each character in the formatted string until the null terminator is reached.
    - If a character is a multi-byte UTF-8 character, it processes it accordingly, checking for overflow against `maxlen`.
    - For regular characters, it checks for special cases like control characters (e.g., `
`, `	`) and writes them to the screen context.
    - The function handles line feeds and carriage returns appropriately, updating the cursor position as needed.
    - Finally, it frees the allocated message buffer before exiting.
- **Output**:
    - The function does not return a value; instead, it writes characters to the screen context, potentially modifying the screen's state based on the input.
- **Functions called**:
    - [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)
    - [`utf8_open`](utf8.c.driver.md#utf8_open)
    - [`utf8_append`](utf8.c.driver.md#utf8_append)
    - [`screen_write_putc`](#screen_write_putc)
    - [`screen_write_cell`](#screen_write_cell)
    - [`screen_write_linefeed`](#screen_write_linefeed)
    - [`screen_write_carriagereturn`](#screen_write_carriagereturn)


---
### screen_write_fast_copy<!-- {{#callable:screen_write_fast_copy}} -->
Copies a rectangular area of cells from a source `screen` to the current `screen` context.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the current screen context.
    - `src`: A pointer to the source `screen` from which cells will be copied.
    - `px`: The x-coordinate of the top-left corner of the rectangle in the source screen.
    - `py`: The y-coordinate of the top-left corner of the rectangle in the source screen.
    - `nx`: The width of the rectangle to copy.
    - `ny`: The height of the rectangle to copy.
- **Control Flow**:
    - Checks if the width (nx) or height (ny) of the rectangle is zero; if so, the function returns immediately.
    - Iterates over each row (yy) from py to py + ny, ensuring it does not exceed the height of the source grid.
    - For each row, it resets the current x-coordinate (cx) to the original value.
    - If a window pane (`wp`) is associated, it initializes the TTY context.
    - Iterates over each column (xx) from px to px + nx, checking if it exceeds the line's cell size.
    - Retrieves the cell at the current position (xx, yy) from the source grid and checks if it fits within the specified width.
    - Sets the cell in the current screen's grid at the current cursor position (cx, cy).
    - If a window pane is associated, it writes the cell to the TTY output.
    - Increments the current x-coordinate (cx) after each cell is processed.
    - After processing all columns in a row, it increments the current y-coordinate (cy) for the next row.
    - Finally, it restores the original cursor position.
- **Output**:
    - The function does not return a value; it directly modifies the current screen's grid by copying cells from the source screen.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)
    - [`grid_get_cell`](grid.c.driver.md#grid_get_cell)
    - [`grid_view_set_cell`](grid-view.c.driver.md#grid_view_set_cell)
    - [`tty_write`](tty.c.driver.md#tty_write)


---
### screen_write_box_border_set<!-- {{#callable:screen_write_box_border_set}} -->
Sets the border style for a box in a grid cell.
- **Inputs**:
    - `lines`: An enumeration value indicating the type of box lines to use (e.g., double, heavy, rounded, etc.).
    - `cell_type`: An integer representing the type of cell for which the border is being set.
    - `gc`: A pointer to a `struct grid_cell` that will be modified to reflect the chosen border style.
- **Control Flow**:
    - The function begins by evaluating the `lines` parameter using a switch statement.
    - If `lines` is `BOX_LINES_NONE`, the function exits without making any changes.
    - For each of the other cases (e.g., `BOX_LINES_DOUBLE`, `BOX_LINES_HEAVY`, etc.), it clears the `GRID_ATTR_CHARSET` attribute from the `gc` structure and sets the `gc->data` to the appropriate border character using the [`utf8_copy`](utf8.c.driver.md#utf8_copy) or [`utf8_set`](utf8.c.driver.md#utf8_set) functions.
    - The function handles multiple border styles by calling different functions to retrieve the appropriate border characters based on the `cell_type`.
- **Output**:
    - The function modifies the provided `grid_cell` structure to set its data to the appropriate border characters based on the specified line style.
- **Functions called**:
    - [`utf8_copy`](utf8.c.driver.md#utf8_copy)
    - [`tty_acs_double_borders`](tty-acs.c.driver.md#tty_acs_double_borders)
    - [`tty_acs_heavy_borders`](tty-acs.c.driver.md#tty_acs_heavy_borders)
    - [`tty_acs_rounded_borders`](tty-acs.c.driver.md#tty_acs_rounded_borders)
    - [`utf8_set`](utf8.c.driver.md#utf8_set)


---
### screen_write_hline<!-- {{#callable:screen_write_hline}} -->
Draws a horizontal line on the screen with specified borders.
- **Inputs**:
    - `ctx`: Pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `nx`: The number of cells to draw the horizontal line across.
    - `left`: An integer indicating whether to draw a left border (1 for yes, 0 for no).
    - `right`: An integer indicating whether to draw a right border (1 for yes, 0 for no).
    - `lines`: An enumeration value indicating the type of box lines to use for the borders.
    - `border_gc`: Pointer to a `grid_cell` structure that defines the border's graphical properties; if NULL, default properties are used.
- **Control Flow**:
    - Retrieve the current cursor position from the screen context.
    - Check if a custom border cell is provided; if not, use the default grid cell.
    - Set the character set attribute for the grid cell.
    - Determine the left border style based on the `left` parameter and set it.
    - Write the left border cell to the screen.
    - Set the middle section of the line to the specified border style and write it for the range from 1 to `nx - 1`.
    - Determine the right border style based on the `right` parameter and set it.
    - Write the right border cell to the screen.
    - Restore the cursor position to its original location.
- **Output**:
    - The function does not return a value; it directly modifies the screen output by drawing a horizontal line with specified borders.
- **Functions called**:
    - [`screen_write_box_border_set`](#screen_write_box_border_set)
    - [`screen_write_cell`](#screen_write_cell)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_vline<!-- {{#callable:screen_write_vline}} -->
The `screen_write_vline` function draws a vertical line on the screen.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `ny`: An unsigned integer representing the height of the vertical line to be drawn.
    - `top`: An integer flag indicating whether to draw a top character ('w' if true, 'x' if false).
    - `bottom`: An integer flag indicating whether to draw a bottom character ('v' if true, 'x' if false).
- **Control Flow**:
    - Retrieve the current cursor position (cx, cy) from the screen context.
    - Copy the default grid cell attributes into a new grid cell structure.
    - Set the character for the top of the vertical line based on the 'top' flag and write it to the screen.
    - Iterate from 1 to ny - 1 to draw the vertical line using the character 'x' at each position.
    - Set the cursor to the bottom position and write the bottom character based on the 'bottom' flag.
    - Reset the cursor position back to the original (cx, cy).
- **Output**:
    - The function does not return a value; it directly modifies the screen output by drawing a vertical line with specified characters at the top and bottom.
- **Functions called**:
    - [`screen_write_putc`](#screen_write_putc)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_menu<!-- {{#callable:screen_write_menu}} -->
The `screen_write_menu` function draws a menu on the screen with specified options and highlights the selected choice.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `menu`: A pointer to a `menu` structure that contains the items to be displayed in the menu.
    - `choice`: An integer representing the index of the currently selected menu item.
    - `lines`: An enumeration value of type `box_lines` that specifies the style of the box lines.
    - `menu_gc`: A pointer to a `grid_cell` structure that defines the graphical characteristics of the menu items.
    - `border_gc`: A pointer to a `grid_cell` structure that defines the graphical characteristics of the border.
    - `choice_gc`: A pointer to a `grid_cell` structure that defines the graphical characteristics of the selected choice.
- **Control Flow**:
    - The function retrieves the current cursor position from the `screen` structure contained in `ctx`.
    - It initializes a default graphical cell by copying the properties from `menu_gc`.
    - A box is drawn around the menu using the [`screen_write_box`](#screen_write_box) function, which takes the menu width and count into account.
    - A loop iterates over each item in the menu, checking if the item name is NULL and drawing a horizontal line if it is.
    - If the current item is the selected choice and is not a separator (indicated by a leading '-'), the graphical cell is set to `choice_gc`.
    - The function then moves the cursor to the appropriate position and fills the menu item area with spaces.
    - If the item name starts with a '-', it is drawn in a dimmed style; otherwise, it is drawn normally.
    - Finally, the cursor is reset to its original position.
- **Output**:
    - The function does not return a value; it directly modifies the screen output by drawing the menu and its items.
- **Functions called**:
    - [`screen_write_box`](#screen_write_box)
    - [`screen_write_cursormove`](#screen_write_cursormove)
    - [`screen_write_hline`](#screen_write_hline)
    - [`screen_write_putc`](#screen_write_putc)
    - [`format_draw`](format-draw.c.driver.md#format_draw)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_box<!-- {{#callable:screen_write_box}} -->
The `screen_write_box` function draws a rectangular box on the screen with specified dimensions, borders, and an optional title.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `nx`: An unsigned integer representing the width of the box.
    - `ny`: An unsigned integer representing the height of the box.
    - `lines`: An enumeration value indicating the style of the box lines (e.g., single, double, etc.).
    - `gcp`: A pointer to a `grid_cell` structure that defines the cell attributes for the box; if NULL, default attributes are used.
    - `title`: A string representing the title to be displayed at the top of the box; if NULL, no title is displayed.
- **Control Flow**:
    - The function retrieves the current cursor position from the `screen` structure.
    - If `gcp` is not NULL, it copies its contents to a local `grid_cell` variable; otherwise, it initializes the local variable with default cell attributes.
    - The function sets the attributes for the box cell to include character set support and disables palette usage.
    - It draws the top border of the box by setting the appropriate border characters and writing them to the screen.
    - It draws the bottom border similarly after moving the cursor to the appropriate position.
    - The sides of the box are drawn by iterating through the height of the box and writing the side cells.
    - If a title is provided, it formats and draws the title within the box.
- **Output**:
    - The function does not return a value; it directly modifies the screen output by drawing the box with the specified attributes and title.
- **Functions called**:
    - [`screen_write_box_border_set`](#screen_write_box_border_set)
    - [`screen_write_cell`](#screen_write_cell)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)
    - [`screen_write_cursormove`](#screen_write_cursormove)
    - [`format_draw`](format-draw.c.driver.md#format_draw)


---
### screen_write_preview<!-- {{#callable:screen_write_preview}} -->
The `screen_write_preview` function writes a preview of a specified area of a source screen to a target screen context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `src`: A pointer to a `screen` structure representing the source screen from which to copy the preview.
    - `nx`: An unsigned integer representing the width of the area to be previewed.
    - `ny`: An unsigned integer representing the height of the area to be previewed.
- **Control Flow**:
    - The function retrieves the current cursor position from the target screen context.
    - It checks if the source screen is in cursor mode; if so, it calculates the preview area based on the cursor's position, ensuring it does not exceed the screen boundaries.
    - If the source screen is not in cursor mode, the preview area defaults to the top-left corner.
    - The function then calls [`screen_write_fast_copy`](#screen_write_fast_copy) to copy the specified area from the source screen to the target screen context.
    - If the source screen is in cursor mode, it retrieves the cell at the cursor position, modifies its attributes to indicate it is selected, and writes it to the target screen context.
- **Output**:
    - The function does not return a value; it modifies the target screen context by writing the previewed area from the source screen.
- **Functions called**:
    - [`screen_write_fast_copy`](#screen_write_fast_copy)
    - [`grid_view_get_cell`](grid-view.c.driver.md#grid_view_get_cell)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)
    - [`screen_write_cell`](#screen_write_cell)


---
### screen_write_mode_set<!-- {{#callable:screen_write_mode_set}} -->
Sets the screen write mode for the given context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `mode`: An integer representing the mode to be set for the screen.
- **Control Flow**:
    - The function retrieves the `screen` structure from the `ctx` context.
    - The specified `mode` is combined with the current mode using a bitwise OR operation.
    - If the logging level is not zero, a debug log is generated to indicate the mode change.
- **Output**:
    - The function does not return a value; it modifies the mode of the screen directly.
- **Functions called**:
    - [`log_get_level`](log.c.driver.md#log_get_level)
    - [`screen_mode_to_string`](screen.c.driver.md#screen_mode_to_string)


---
### screen_write_mode_clear<!-- {{#callable:screen_write_mode_clear}} -->
Clears a specified mode from the screen context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `mode`: An integer representing the mode to be cleared from the screen.
- **Control Flow**:
    - Retrieve the `screen` structure from the `ctx` context.
    - Use a bitwise AND operation to clear the specified `mode` from the `screen`'s current modes.
    - Check if the logging level is not zero.
    - If logging is enabled, log a debug message indicating the mode that was cleared.
- **Output**:
    - The function does not return a value; it modifies the state of the `screen` by clearing the specified mode.
- **Functions called**:
    - [`log_get_level`](log.c.driver.md#log_get_level)
    - [`screen_mode_to_string`](screen.c.driver.md#screen_mode_to_string)


---
### screen_write_cursorup<!-- {{#callable:screen_write_cursorup}} -->
Moves the cursor up by a specified number of lines.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `ny`: An unsigned integer representing the number of lines to move the cursor up.
- **Control Flow**:
    - If `ny` is zero, it is set to one to ensure at least one line is moved.
    - The function checks if the current cursor position `cy` is above the upper scroll region `rupper`.
    - If `cy` is above `rupper`, it limits `ny` to the current cursor position to prevent moving above the top of the screen.
    - If `cy` is below `rupper`, it limits `ny` to the difference between `cy` and `rupper`.
    - The cursor's x-coordinate `cx` is adjusted if it is at the rightmost edge of the screen.
    - The cursor's y-coordinate `cy` is decremented by `ny` to move it up.
    - Finally, the [`screen_write_set_cursor`](#screen_write_set_cursor) function is called to update the cursor position.
- **Output**:
    - The function does not return a value; it directly modifies the cursor position in the screen context.
- **Functions called**:
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_cursordown<!-- {{#callable:screen_write_cursordown}} -->
Moves the cursor down by a specified number of lines.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `ny`: An unsigned integer representing the number of lines to move the cursor down.
- **Control Flow**:
    - If `ny` is zero, it is set to one to ensure at least one line is moved.
    - The function checks if the current cursor position is above or below the defined scroll region (`rupper` and `rlower`).
    - If the cursor is below the `rlower`, it limits `ny` to the maximum number of lines that can be moved down without exceeding the screen size.
    - If the cursor is above the `rupper`, it limits `ny` to the maximum number of lines that can be moved down without exceeding the scroll region.
    - If the cursor is at the rightmost column, it decrements the column index (`cx`) to avoid overflow.
    - The cursor's vertical position (`cy`) is incremented by `ny` to move it down.
    - Finally, the function calls [`screen_write_set_cursor`](#screen_write_set_cursor) to update the cursor's position.
- **Output**:
    - The function does not return a value; it modifies the cursor position within the screen context.
- **Functions called**:
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_cursorright<!-- {{#callable:screen_write_cursorright}} -->
Moves the cursor to the right by a specified number of positions.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `nx`: An unsigned integer representing the number of positions to move the cursor to the right.
- **Control Flow**:
    - If `nx` is zero, it is set to one to ensure at least one position is moved.
    - The function checks if moving `nx` positions exceeds the screen's width; if so, it adjusts `nx` to fit within the screen.
    - If `nx` is still zero after adjustments, the function returns without making any changes.
    - The current cursor position `cx` is incremented by `nx`.
    - The new cursor position is set using the [`screen_write_set_cursor`](#screen_write_set_cursor) function.
- **Output**:
    - The function does not return a value; it modifies the cursor position on the screen based on the input parameters.
- **Functions called**:
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_cursorleft<!-- {{#callable:screen_write_cursorleft}} -->
Moves the cursor left by a specified number of positions.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `nx`: An unsigned integer representing the number of positions to move the cursor left.
- **Control Flow**:
    - If `nx` is zero, it is set to one to ensure at least one position is moved.
    - If `nx` exceeds the current cursor position (`cx`), it is adjusted to equal `cx`.
    - If `nx` is still zero after adjustments, the function returns without making any changes.
    - The cursor's x-coordinate (`cx`) is decremented by `nx`.
    - The [`screen_write_set_cursor`](#screen_write_set_cursor) function is called to update the cursor's position.
- **Output**:
    - The function does not return a value; it modifies the cursor position on the screen based on the input parameters.
- **Functions called**:
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_backspace<!-- {{#callable:screen_write_backspace}} -->
The `screen_write_backspace` function moves the cursor left by one position, handling line wrapping appropriately.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing, including the current screen state.
- **Control Flow**:
    - The function retrieves the current cursor position (`cx`, `cy`) from the `screen` structure within the context.
    - If the cursor is at the beginning of the line (`cx == 0`), it checks if the cursor is also at the top of the screen (`cy == 0`). If so, it returns without making any changes.
    - If the cursor is at the beginning of the line but not at the top, it retrieves the previous line's grid line and checks if it is wrapped.
    - If the previous line is wrapped, it moves the cursor up to the previous line and sets the cursor to the last column of that line.
    - If the cursor is not at the beginning of the line, it simply decrements the cursor's x-coordinate (`cx`).
    - Finally, it calls [`screen_write_set_cursor`](#screen_write_set_cursor) to update the cursor position in the context.
- **Output**:
    - The function does not return a value; it modifies the cursor position in the screen context based on the backspace operation.
- **Functions called**:
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_alignmenttest<!-- {{#callable:screen_write_alignmenttest}} -->
The `screen_write_alignmenttest` function performs a VT100 alignment test by filling the screen with a specific character and sending a command to the terminal.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing operations.
- **Control Flow**:
    - The function begins by copying the default grid cell into a local variable `gc` and setting its data to the character 'E'.
    - If the `ENABLE_SIXEL` macro is defined, it checks if there are any free images and sets the redraw flag for the window pane if applicable.
    - It then iterates over each cell in the screen's grid, setting each cell to the character 'E'.
    - After filling the grid, it resets the cursor position to the top-left corner of the screen.
    - The function sets the screen's upper and lower scroll regions.
    - It initializes the TTY context for writing and collects commands to clear the screen.
    - Finally, it sends the alignment test command to the terminal.
- **Output**:
    - The function does not return a value but modifies the screen's grid and sends a command to the terminal to perform an alignment test, visually represented by filling the screen with the character 'E'.
- **Functions called**:
    - [`utf8_set`](utf8.c.driver.md#utf8_set)
    - [`image_free_all`](image.c.driver.md#image_free_all)
    - [`grid_view_set_cell`](grid-view.c.driver.md#grid_view_set_cell)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`screen_write_collect_clear`](#screen_write_collect_clear)
    - [`tty_write`](tty.c.driver.md#tty_write)


---
### screen_write_insertcharacter<!-- {{#callable:screen_write_insertcharacter}} -->
Inserts a specified number of characters at the current cursor position in the screen context.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `nx`: An unsigned integer representing the number of characters to insert.
    - `bg`: An unsigned integer representing the background color to use for the inserted characters.
- **Control Flow**:
    - If `nx` is zero, it is set to one to ensure at least one character is inserted.
    - If `nx` exceeds the available space to the right of the cursor, it is adjusted to fit within the screen width.
    - If `nx` is still zero after adjustments, the function returns without making any changes.
    - If the cursor's x-coordinate exceeds the screen width, the function returns without making any changes.
    - If the `ENABLE_SIXEL` feature is enabled and the current line requires a redraw, the pane's redraw flag is set.
    - The screen writing context is initialized for the operation.
    - The grid is updated to insert the specified number of cells at the current cursor position with the specified background color.
    - The collected screen write operations are flushed to ensure they are executed.
    - The terminal is instructed to insert the specified number of characters using the appropriate TTY command.
- **Output**:
    - The function does not return a value; it modifies the screen's grid and sends commands to the terminal to reflect the inserted characters.
- **Functions called**:
    - [`image_check_line`](image.c.driver.md#image_check_line)
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`grid_view_insert_cells`](grid-view.c.driver.md#grid_view_insert_cells)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`tty_write`](tty.c.driver.md#tty_write)


---
### screen_write_deletecharacter<!-- {{#callable:screen_write_deletecharacter}} -->
Deletes a specified number of characters from the current cursor position in the screen context.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `nx`: An unsigned integer representing the number of characters to delete.
    - `bg`: An unsigned integer representing the background color to use for the deleted characters.
- **Control Flow**:
    - If `nx` is zero, it is set to one to ensure at least one character is deleted.
    - If `nx` exceeds the number of characters available from the current cursor position to the end of the line, it is adjusted to that maximum.
    - If `nx` is still zero after adjustments, the function returns immediately.
    - If the cursor's x-coordinate is out of bounds, the function returns immediately.
    - If the `ENABLE_SIXEL` feature is enabled and the current line has an image, the pane is marked for redraw.
    - The screen writing context is initialized for the operation.
    - The [`grid_view_delete_cells`](grid-view.c.driver.md#grid_view_delete_cells) function is called to perform the actual deletion of characters from the grid.
    - The collected changes are flushed to the output, and a TTY command to delete characters is sent.
- **Output**:
    - The function does not return a value; it modifies the screen's grid and sends commands to the terminal to reflect the deletion of characters.
- **Functions called**:
    - [`image_check_line`](image.c.driver.md#image_check_line)
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`grid_view_delete_cells`](grid-view.c.driver.md#grid_view_delete_cells)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`tty_write`](tty.c.driver.md#tty_write)


---
### screen_write_clearcharacter<!-- {{#callable:screen_write_clearcharacter}} -->
Clears a specified number of characters on the screen from the current cursor position.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `nx`: An unsigned integer representing the number of characters to clear.
    - `bg`: An unsigned integer representing the background color to use when clearing the characters.
- **Control Flow**:
    - If `nx` is zero, it is set to one to ensure at least one character is cleared.
    - If `nx` exceeds the available space to the right of the cursor, it is adjusted to fit within the screen bounds.
    - If `nx` is still zero after adjustments, the function returns early.
    - If the cursor's x-coordinate exceeds the screen width, the function returns early.
    - If the `ENABLE_SIXEL` feature is enabled and the current line requires redrawing, the pane's redraw flag is set.
    - The screen writing context is initialized for the operation.
    - The grid is updated to clear the specified characters at the current cursor position with the specified background color.
    - The collected screen write operations are flushed to the output.
    - The terminal is instructed to clear the specified number of characters.
- **Output**:
    - The function does not return a value; it performs operations to clear characters on the screen and updates the terminal display accordingly.
- **Functions called**:
    - [`image_check_line`](image.c.driver.md#image_check_line)
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`grid_view_clear`](grid-view.c.driver.md#grid_view_clear)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`tty_write`](tty.c.driver.md#tty_write)


---
### screen_write_insertline<!-- {{#callable:screen_write_insertline}} -->
Inserts a specified number of lines at the current cursor position in a screen context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `ny`: An unsigned integer representing the number of lines to insert.
    - `bg`: An unsigned integer representing the background color to use for the inserted lines.
- **Control Flow**:
    - If `ny` is zero, it is set to one to ensure at least one line is inserted.
    - If the current cursor position is outside the defined scroll region, the function checks if the number of lines to insert exceeds the available space and adjusts `ny` accordingly.
    - The function initializes a `tty_ctx` structure for terminal context and sets the background color.
    - It calls [`grid_view_insert_lines`](grid-view.c.driver.md#grid_view_insert_lines) to perform the actual insertion of lines in the grid.
    - The function collects any pending output and flushes it, then sends a command to the terminal to insert the specified number of lines.
- **Output**:
    - The function does not return a value; it modifies the screen's grid and sends commands to the terminal to reflect the changes.
- **Functions called**:
    - [`image_check_line`](image.c.driver.md#image_check_line)
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`grid_view_insert_lines`](grid-view.c.driver.md#grid_view_insert_lines)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`tty_write`](tty.c.driver.md#tty_write)
    - [`grid_view_insert_lines_region`](grid-view.c.driver.md#grid_view_insert_lines_region)


---
### screen_write_deleteline<!-- {{#callable:screen_write_deleteline}} -->
Deletes a specified number of lines from the screen at the current cursor position.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `ny`: An unsigned integer representing the number of lines to delete.
    - `bg`: An unsigned integer representing the background color to use when deleting lines.
- **Control Flow**:
    - If `ny` is zero, it is set to one to ensure at least one line is deleted.
    - If the current cursor position (`s->cy`) is outside the defined scroll region (`s->rupper` and `s->rlower`), it checks if the number of lines to delete exceeds the available lines and adjusts accordingly.
    - The function initializes the TTY context and sets the background color.
    - It calls [`grid_view_delete_lines`](grid-view.c.driver.md#grid_view_delete_lines) to delete the specified number of lines from the grid.
    - The function collects and flushes the changes to the screen context.
    - Finally, it sends a command to the TTY to execute the deletion of lines.
- **Output**:
    - The function does not return a value; it modifies the screen by deleting the specified number of lines and updates the TTY accordingly.
- **Functions called**:
    - [`image_check_line`](image.c.driver.md#image_check_line)
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`grid_view_delete_lines`](grid-view.c.driver.md#grid_view_delete_lines)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`tty_write`](tty.c.driver.md#tty_write)
    - [`grid_view_delete_lines_region`](grid-view.c.driver.md#grid_view_delete_lines_region)


---
### screen_write_clearline<!-- {{#callable:screen_write_clearline}} -->
Clears the entire line at the current cursor position in the screen context.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `bg`: An unsigned integer representing the background color to be used when clearing the line.
- **Control Flow**:
    - Retrieve the current screen from the context.
    - Get the grid line at the current cursor Y position.
    - Check if the line is empty and if the background color is the default; if so, return early.
    - If the `ENABLE_SIXEL` feature is enabled, check if the line needs to be redrawn and set the redraw flag if necessary.
    - Clear the specified line in the grid with the given background color.
    - Collect the clear operation for the current line.
    - Update the current item in the context with the clear operation details.
    - Insert the clear operation into the write list for the current line.
- **Output**:
    - The function does not return a value; it modifies the screen context by clearing the specified line and updating the write list.
- **Functions called**:
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)
    - [`image_check_line`](image.c.driver.md#image_check_line)
    - [`grid_view_clear`](grid-view.c.driver.md#grid_view_clear)
    - [`screen_write_collect_clear`](#screen_write_collect_clear)
    - [`screen_write_get_citem`](#screen_write_get_citem)


---
### screen_write_clearendofline<!-- {{#callable:screen_write_clearendofline}} -->
Clears the line from the cursor to the end of the line in the terminal screen.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `bg`: An unsigned integer representing the background color to be used for clearing.
- **Control Flow**:
    - Checks if the cursor is at the beginning of the line; if so, it calls [`screen_write_clearline`](#screen_write_clearline) to clear the entire line.
    - Retrieves the current line from the grid using [`grid_get_line`](grid.c.driver.md#grid_get_line).
    - Validates the cursor position to ensure it is within the screen bounds and not beyond the end of the line.
    - If the cursor is valid, it clears the cells from the cursor position to the end of the line using [`grid_view_clear`](grid-view.c.driver.md#grid_view_clear).
    - Collects the clear operation for later processing by calling [`screen_write_collect_trim`](#screen_write_collect_trim).
    - Updates the context item with the new clear operation details and inserts it into the write list.
- **Output**:
    - The function does not return a value but modifies the screen's grid to reflect the cleared area and updates the context for future operations.
- **Functions called**:
    - [`screen_write_clearline`](#screen_write_clearline)
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)
    - [`image_check_line`](image.c.driver.md#image_check_line)
    - [`grid_view_clear`](grid-view.c.driver.md#grid_view_clear)
    - [`screen_write_collect_trim`](#screen_write_collect_trim)
    - [`screen_write_get_citem`](#screen_write_get_citem)


---
### screen_write_clearstartofline<!-- {{#callable:screen_write_clearstartofline}} -->
Clears the start of the current line on the screen.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `bg`: An unsigned integer representing the background color to be used for clearing.
- **Control Flow**:
    - Checks if the current cursor position is at or beyond the last column of the screen; if so, it calls [`screen_write_clearline`](#screen_write_clearline) to clear the entire line and returns.
    - If the cursor position is valid, it checks if the `ENABLE_SIXEL` feature is enabled and marks the pane for redraw if necessary.
    - Clears the grid cells from the start of the line to the current cursor position, using the specified background color.
    - Collects the cleared area into a `screen_write_citem` structure, setting its properties such as position and type.
    - Inserts the collected item into the appropriate position in the write list for the current line.
- **Output**:
    - The function does not return a value; it modifies the screen's state by clearing part of the line and updating the write list.
- **Functions called**:
    - [`screen_write_clearline`](#screen_write_clearline)
    - [`image_check_line`](image.c.driver.md#image_check_line)
    - [`grid_view_clear`](grid-view.c.driver.md#grid_view_clear)
    - [`screen_write_collect_trim`](#screen_write_collect_trim)
    - [`screen_write_get_citem`](#screen_write_get_citem)


---
### screen_write_cursormove<!-- {{#callable:screen_write_cursormove}} -->
Moves the cursor to a specified position on the screen.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `px`: The x-coordinate (column) to move the cursor to.
    - `py`: The y-coordinate (row) to move the cursor to.
    - `origin`: An integer indicating whether to adjust the y-coordinate based on the origin mode.
- **Control Flow**:
    - Checks if the origin mode is enabled and adjusts the y-coordinate if necessary.
    - Validates the x-coordinate and y-coordinate against the screen dimensions, ensuring they do not exceed the screen boundaries.
    - Logs the cursor movement from the current position to the new position.
    - Calls [`screen_write_set_cursor`](#screen_write_set_cursor) to actually move the cursor to the new position.
- **Output**:
    - The function does not return a value; it modifies the cursor position on the screen based on the provided coordinates.
- **Functions called**:
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_reverseindex<!-- {{#callable:screen_write_reverseindex}} -->
The `screen_write_reverseindex` function scrolls the screen up by one line if the cursor is at the top of the scroll region.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `bg`: An unsigned integer representing the background color to be used during the operation.
- **Control Flow**:
    - Checks if the cursor's y-coordinate (`cy`) is at the upper limit of the scroll region (`rupper`).
    - If at the upper limit, it checks if any images need to be freed and sets the redraw flag if necessary.
    - Calls [`grid_view_scroll_region_down`](grid-view.c.driver.md#grid_view_scroll_region_down) to scroll the visible region down by one line.
    - Flushes any collected screen write operations using [`screen_write_collect_flush`](#screen_write_collect_flush).
    - Initializes the TTY context and sets the background color.
    - Sends a command to the TTY to perform a reverse index operation.
    - If the cursor is not at the upper limit but above the first line, it moves the cursor up by one line.
- **Output**:
    - The function does not return a value but modifies the screen's state by scrolling the content and potentially updating the cursor position.
- **Functions called**:
    - [`image_free_all`](image.c.driver.md#image_free_all)
    - [`grid_view_scroll_region_down`](grid-view.c.driver.md#grid_view_scroll_region_down)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`tty_write`](tty.c.driver.md#tty_write)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_scrollregion<!-- {{#callable:screen_write_scrollregion}} -->
Sets the scroll region for the screen.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `rupper`: An unsigned integer representing the upper boundary of the scroll region.
    - `rlower`: An unsigned integer representing the lower boundary of the scroll region.
- **Control Flow**:
    - Checks if `rupper` exceeds the maximum screen height and adjusts it accordingly.
    - Checks if `rlower` exceeds the maximum screen height and adjusts it accordingly.
    - Validates that `rupper` is less than `rlower` to ensure a valid scroll region.
    - Flushes any collected screen write operations.
    - Sets the cursor position to the top-left corner of the screen.
    - Updates the screen's `rupper` and `rlower` properties to define the new scroll region.
- **Output**:
    - The function does not return a value; it modifies the scroll region of the screen directly.
- **Functions called**:
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_linefeed<!-- {{#callable:screen_write_linefeed}} -->
The `screen_write_linefeed` function handles the line feed operation in a terminal screen context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `wrapped`: An integer indicating whether the line is wrapped.
    - `bg`: An unsigned integer representing the background color.
- **Control Flow**:
    - Retrieve the current screen and grid from the context.
    - If the line is wrapped, set the corresponding flag in the grid line.
    - Log the current cursor position and the screen region.
    - If the background color has changed, flush the collected output and update the background color in the context.
    - Check if the cursor is at the bottom of the screen region.
    - If at the bottom, scroll the region up and collect the scroll operation.
    - If not at the bottom, move the cursor down to the next line.
- **Output**:
    - The function does not return a value but modifies the screen context and grid state based on the line feed operation.
- **Functions called**:
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`image_scroll_up`](image.c.driver.md#image_scroll_up)
    - [`image_check_line`](image.c.driver.md#image_check_line)
    - [`grid_view_scroll_region_up`](grid-view.c.driver.md#grid_view_scroll_region_up)
    - [`screen_write_collect_scroll`](#screen_write_collect_scroll)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_scrollup<!-- {{#callable:screen_write_scrollup}} -->
The `screen_write_scrollup` function scrolls the visible region of a terminal screen up by a specified number of lines.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `lines`: An unsigned integer representing the number of lines to scroll up.
    - `bg`: An unsigned integer representing the background color to use during the scroll.
- **Control Flow**:
    - If `lines` is zero, it defaults to one line.
    - If `lines` exceeds the scrollable region, it is capped to the maximum scrollable lines.
    - If the background color `bg` differs from the current context's background, the function flushes the current context and updates the background color.
    - If the `ENABLE_SIXEL` feature is enabled and the image scrolls up successfully, it marks the window pane for redraw.
    - The function then iterates over the number of lines to scroll, calling [`grid_view_scroll_region_up`](grid-view.c.driver.md#grid_view_scroll_region_up) to perform the actual scrolling for each line.
    - After scrolling, it collects the scroll operation for the context.
- **Output**:
    - The function does not return a value but modifies the screen's visible content by scrolling it up and updating the context.
- **Functions called**:
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`image_scroll_up`](image.c.driver.md#image_scroll_up)
    - [`grid_view_scroll_region_up`](grid-view.c.driver.md#grid_view_scroll_region_up)
    - [`screen_write_collect_scroll`](#screen_write_collect_scroll)


---
### screen_write_scrolldown<!-- {{#callable:screen_write_scrolldown}} -->
The `screen_write_scrolldown` function scrolls the visible region of a terminal screen down by a specified number of lines.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `lines`: An unsigned integer representing the number of lines to scroll down.
    - `bg`: An unsigned integer representing the background color to use during the scroll.
- **Control Flow**:
    - The function initializes the TTY context using [`screen_write_initctx`](#screen_write_initctx) with the provided `ctx` and sets the background color.
    - If `lines` is zero, it defaults to scrolling down by one line; if it exceeds the scrollable region, it is capped to the maximum scrollable lines.
    - If the `ENABLE_SIXEL` feature is enabled, it checks if any images need to be freed and marks the pane for redraw if necessary.
    - The function then enters a loop to scroll the grid down by the specified number of lines using [`grid_view_scroll_region_down`](grid-view.c.driver.md#grid_view_scroll_region_down).
    - After scrolling, it collects any changes made during the operation and flushes them to the TTY using [`screen_write_collect_flush`](#screen_write_collect_flush).
- **Output**:
    - The function does not return a value but modifies the screen's display by scrolling the visible region down and updating the TTY output accordingly.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`image_free_all`](image.c.driver.md#image_free_all)
    - [`grid_view_scroll_region_down`](grid-view.c.driver.md#grid_view_scroll_region_down)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`tty_write`](tty.c.driver.md#tty_write)


---
### screen_write_carriagereturn<!-- {{#callable:screen_write_carriagereturn}} -->
The `screen_write_carriagereturn` function moves the cursor to the beginning of the current line.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing operations.
- **Control Flow**:
    - The function calls [`screen_write_set_cursor`](#screen_write_set_cursor) with parameters 0 and -1.
    - The [`screen_write_set_cursor`](#screen_write_set_cursor) function updates the cursor's x-coordinate to 0 (the start of the line) and leaves the y-coordinate unchanged (indicated by -1).
- **Output**:
    - The function does not return a value; it modifies the cursor position in the provided context.
- **Functions called**:
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_clearendofscreen<!-- {{#callable:screen_write_clearendofscreen}} -->
Clears the screen from the current cursor position to the end.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `bg`: An unsigned integer representing the background color to be used for clearing.
- **Control Flow**:
    - Checks if the current cursor position is at the top-left corner of the screen and if history scrolling is enabled.
    - If conditions are met, it clears the history grid.
    - Otherwise, it clears the grid from the current cursor position to the end of the line.
    - Then, it clears the grid from the next line down to the bottom of the screen.
    - Collects the clear operation for later processing.
    - Flushes the collected operations to the terminal.
- **Output**:
    - The function does not return a value; it performs operations to clear the screen and update the display accordingly.
- **Functions called**:
    - [`image_check_line`](image.c.driver.md#image_check_line)
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`grid_view_clear_history`](grid-view.c.driver.md#grid_view_clear_history)
    - [`grid_view_clear`](grid-view.c.driver.md#grid_view_clear)
    - [`screen_write_collect_clear`](#screen_write_collect_clear)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`tty_write`](tty.c.driver.md#tty_write)


---
### screen_write_clearstartofscreen<!-- {{#callable:screen_write_clearstartofscreen}} -->
Clears the screen from the start to the current cursor position.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `bg`: An unsigned integer representing the background color to be used for clearing.
- **Control Flow**:
    - The function retrieves the current screen from the context.
    - It checks if the `ENABLE_SIXEL` feature is enabled and if the image needs to be redrawn.
    - The function initializes the TTY context with the specified background color.
    - It clears the grid from the top of the screen to the current cursor position, handling both the current line and any lines above it.
    - It collects the clear operation for later processing.
    - Finally, it sends a command to the TTY to clear the start of the screen.
- **Output**:
    - The function does not return a value; it performs operations to clear the screen and update the display accordingly.
- **Functions called**:
    - [`image_check_line`](image.c.driver.md#image_check_line)
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`grid_view_clear`](grid-view.c.driver.md#grid_view_clear)
    - [`screen_write_collect_clear`](#screen_write_collect_clear)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`tty_write`](tty.c.driver.md#tty_write)


---
### screen_write_clearscreen<!-- {{#callable:screen_write_clearscreen}} -->
Clears the entire screen and optionally scrolls the history.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `bg`: An unsigned integer representing the background color to be used when clearing the screen.
- **Control Flow**:
    - Checks if the `ENABLE_SIXEL` feature is enabled and if so, calls [`image_free_all`](image.c.driver.md#image_free_all) to free any images and sets the redraw flag for the window pane if necessary.
    - Initializes the `tty_ctx` structure with the background color.
    - Checks if the screen's grid has the `GRID_HISTORY` flag set and if the option `scroll-on-clear` is enabled; if so, it calls [`grid_view_clear_history`](grid-view.c.driver.md#grid_view_clear_history) to clear the history.
    - If history clearing is not enabled, it calls [`grid_view_clear`](grid-view.c.driver.md#grid_view_clear) to clear the entire screen grid.
    - Calls [`screen_write_collect_clear`](#screen_write_collect_clear) to collect the clear operation for the current context.
    - Finally, sends the clear screen command to the terminal using [`tty_write`](tty.c.driver.md#tty_write).
- **Output**:
    - The function does not return a value; it performs operations to clear the screen and update the display accordingly.
- **Functions called**:
    - [`image_free_all`](image.c.driver.md#image_free_all)
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`grid_view_clear_history`](grid-view.c.driver.md#grid_view_clear_history)
    - [`grid_view_clear`](grid-view.c.driver.md#grid_view_clear)
    - [`screen_write_collect_clear`](#screen_write_collect_clear)
    - [`tty_write`](tty.c.driver.md#tty_write)


---
### screen_write_clearhistory<!-- {{#callable:screen_write_clearhistory}} -->
Clears the entire history of the screen.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls [`grid_clear_history`](grid.c.driver.md#grid_clear_history) with the grid associated with the screen context.
    - No conditional logic or loops are present in this function.
- **Output**:
    - The function does not return a value; it performs an action that clears the screen's history.
- **Functions called**:
    - [`grid_clear_history`](grid.c.driver.md#grid_clear_history)


---
### screen_write_fullredraw<!-- {{#callable:screen_write_fullredraw}} -->
The `screen_write_fullredraw` function forces a complete redraw of the screen context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
- **Control Flow**:
    - Calls [`screen_write_collect_flush`](#screen_write_collect_flush) to flush any pending screen write operations.
    - Initializes a `tty_ctx` structure for the current screen context.
    - If the `redraw_cb` callback in the `tty_ctx` is not NULL, it invokes this callback, passing the `tty_ctx` as an argument.
- **Output**:
    - The function does not return a value; it performs actions that result in the screen being redrawn based on the current context.
- **Functions called**:
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_write_initctx`](#screen_write_initctx)


---
### screen_write_collect_trim<!-- {{#callable:screen_write_collect_trim}} -->
The `screen_write_collect_trim` function trims collected screen write items based on specified coordinates.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `y`: An unsigned integer representing the row index in the screen write list.
    - `x`: An unsigned integer representing the starting column index for trimming.
    - `used`: An unsigned integer indicating the number of columns to trim.
    - `wrapped`: A pointer to an integer that indicates if the trimmed item was wrapped.
- **Control Flow**:
    - Checks if the list of items for the specified row is empty and returns NULL if it is.
    - Iterates through each item in the list using a safe loop to avoid issues during removal.
    - For each item, it checks its position relative to the specified trim coordinates (before, after, inside, etc.).
    - If an item is entirely before the trim range, it continues to the next item.
    - If an item is entirely after the trim range, it sets the 'before' pointer and breaks the loop.
    - If an item is entirely inside the trim range, it removes the item from the list and frees it.
    - If an item overlaps the start of the trim range, it adjusts the item's used length.
    - If an item overlaps the end of the trim range, it adjusts the item's starting position and used length.
    - If an item covers both sides of the trim range, it splits the item into two and inserts the new item into the list.
- **Output**:
    - Returns a pointer to the item that was before the trimmed range, or NULL if no such item exists.
- **Functions called**:
    - [`screen_write_free_citem`](#screen_write_free_citem)
    - [`screen_write_get_citem`](#screen_write_get_citem)


---
### screen_write_collect_clear<!-- {{#callable:screen_write_collect_clear}} -->
The `screen_write_collect_clear` function clears collected screen write items for a specified range of lines.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `y`: The starting line number from which to begin clearing collected items.
    - `n`: The number of lines to clear starting from line `y`.
- **Control Flow**:
    - Iterates from line `y` to `y + n`, inclusive.
    - For each line, retrieves the corresponding `screen_write_cline` structure from the context's write list.
    - Concatenates the items of the current line's item list to a free list, effectively clearing them.
- **Output**:
    - The function does not return a value; it modifies the internal state of the `screen_write_ctx` by clearing the collected items for the specified lines.


---
### screen_write_collect_scroll<!-- {{#callable:screen_write_collect_scroll}} -->
The `screen_write_collect_scroll` function handles the scrolling of a specified region in the screen context.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `bg`: An unsigned integer representing the background color to be used during the scroll operation.
- **Control Flow**:
    - Logs the current cursor position and the scroll region boundaries.
    - Calls [`screen_write_collect_clear`](#screen_write_collect_clear) to clear the current line at the upper boundary of the scroll region.
    - Saves the data of the upper boundary line to a temporary variable.
    - Iterates from the upper boundary to the lower boundary of the scroll region, concatenating the items from the next line into the current line.
    - Sets the data of the lower boundary line to the saved data from the upper boundary.
    - Creates a new `screen_write_citem` for the clear operation at the lower boundary and inserts it into the items list.
- **Output**:
    - The function does not return a value but modifies the screen context by updating the lines in the specified scroll region and preparing for a clear operation at the bottom.
- **Functions called**:
    - [`screen_write_collect_clear`](#screen_write_collect_clear)
    - [`screen_write_get_citem`](#screen_write_get_citem)


---
### screen_write_collect_flush<!-- {{#callable:screen_write_collect_flush}} -->
The `screen_write_collect_flush` function processes and flushes collected screen write items to the terminal.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `scroll_only`: An integer flag indicating whether to only perform scrolling without flushing other items.
    - `from`: A string indicating the source or context from which this function was called, used for logging.
- **Control Flow**:
    - Checks if there are any scrolled items and logs the scrolling action if necessary.
    - Initializes the TTY context for scrolling and performs the scroll operation if required.
    - Resets the background color and checks if only scrolling is requested, returning early if so.
    - Iterates over each line of the screen, processing each collected item in the write list.
    - For each item, sets the cursor position, determines the type of item (clear or character), and sends the appropriate command to the terminal.
    - Removes processed items from the collection and frees their memory.
    - Restores the cursor position to its original state after processing all items.
- **Output**:
    - The function does not return a value but flushes the collected screen write items to the terminal, updating the display accordingly.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`tty_write`](tty.c.driver.md#tty_write)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)
    - [`screen_write_free_citem`](#screen_write_free_citem)


---
### screen_write_collect_end<!-- {{#callable:screen_write_collect_end}} -->
The `screen_write_collect_end` function finalizes the collection of screen write items and updates the screen accordingly.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing operations.
- **Control Flow**:
    - Checks if the `used` field of the current item is zero; if so, it returns immediately.
    - Calls [`screen_write_collect_trim`](#screen_write_collect_trim) to adjust the collected items based on the current cursor position and the number of used characters.
    - Updates the `x` position and `wrapped` status of the current item based on the cursor position.
    - Inserts the current item into the appropriate position in the linked list of collected items.
    - Logs the details of the collected item for debugging purposes.
    - Clears any padding cells to the left of the current cursor position if necessary.
    - Sets the cells in the grid with the collected character data.
    - Updates the cursor position to the end of the newly written characters.
    - Clears any padding cells to the right of the current cursor position.
- **Output**:
    - The function does not return a value but modifies the screen's grid by updating the cells with the collected character data and adjusting the cursor position.
- **Functions called**:
    - [`screen_write_collect_trim`](#screen_write_collect_trim)
    - [`screen_write_get_citem`](#screen_write_get_citem)
    - [`grid_view_get_cell`](grid-view.c.driver.md#grid_view_get_cell)
    - [`grid_view_set_cell`](grid-view.c.driver.md#grid_view_set_cell)
    - [`image_check_area`](image.c.driver.md#image_check_area)
    - [`grid_view_set_cells`](grid-view.c.driver.md#grid_view_set_cells)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_collect_add<!-- {{#callable:screen_write_collect_add}} -->
The `screen_write_collect_add` function adds a character to the current screen context, collecting it if certain conditions are met.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the current screen context.
    - `gc`: A pointer to the `grid_cell` structure representing the character and its attributes to be added.
- **Control Flow**:
    - The function first checks if the character can be collected based on its attributes and the current screen state.
    - If the character cannot be collected, it ends the current collection, flushes any collected data, and writes the character directly to the screen.
    - If the character can be collected, it checks if the current cursor position exceeds the screen width and ends the collection if necessary.
    - If the cursor position is valid, it updates the collected item with the character data and increments the used count.
- **Output**:
    - The function does not return a value; instead, it modifies the screen context by either adding the character to the collection or writing it directly to the screen.
- **Functions called**:
    - [`screen_write_collect_end`](#screen_write_collect_end)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_write_cell`](#screen_write_cell)
    - [`screen_write_linefeed`](#screen_write_linefeed)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)


---
### screen_write_cell<!-- {{#callable:screen_write_cell}} -->
The `screen_write_cell` function writes a grid cell to the screen, handling various conditions such as wrapping, combining characters, and selection.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `gc`: A pointer to a `grid_cell` structure that represents the cell to be written to the screen.
- **Control Flow**:
    - Checks if the cell is a padding cell and returns early if true.
    - Attempts to combine the current cell with the previous one if applicable.
    - Flushes any existing scrolling before writing the new cell.
    - Checks if the character fits within the current line and handles wrapping if necessary.
    - In insert mode, makes space for the new cell.
    - Sanity checks the cursor position to ensure it is within bounds.
    - Handles overwriting of existing UTF-8 characters if necessary.
    - Fills in padding cells if the new character is wide.
    - Determines if the new cell is different from the existing one and updates the grid accordingly.
    - Updates the selected flag for the cell if it is part of a selection.
    - Moves the cursor to the appropriate position after writing the cell.
    - Writes the cell to the screen using the appropriate TTY command.
- **Output**:
    - The function does not return a value but updates the screen's grid with the new cell data and manages the cursor position accordingly.
- **Functions called**:
    - [`screen_write_combine`](#screen_write_combine)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`grid_view_insert_cells`](grid-view.c.driver.md#grid_view_insert_cells)
    - [`screen_write_linefeed`](#screen_write_linefeed)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)
    - [`grid_view_get_cell`](grid-view.c.driver.md#grid_view_get_cell)
    - [`screen_write_overwrite`](#screen_write_overwrite)
    - [`grid_view_set_padding`](grid-view.c.driver.md#grid_view_set_padding)
    - [`grid_cells_equal`](grid.c.driver.md#grid_cells_equal)
    - [`screen_check_selection`](screen.c.driver.md#screen_check_selection)
    - [`grid_view_set_cell`](grid-view.c.driver.md#grid_view_set_cell)
    - [`tty_write`](tty.c.driver.md#tty_write)
    - [`screen_select_cell`](screen.c.driver.md#screen_select_cell)


---
### screen_write_combine<!-- {{#callable:screen_write_combine}} -->
Combines a UTF-8 character with the previous character in the grid if applicable.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `gc`: A pointer to the `grid_cell` structure representing the character to be combined.
- **Control Flow**:
    - Checks if the character is a zero-width joiner (ZWJ) or a variation selector (VS) and sets flags accordingly.
    - Returns early if the character cannot be combined due to its size or position.
    - Retrieves the last character in the grid to determine if combination is possible.
    - Checks if the last character can be combined with the current character based on width and padding.
    - If combining is valid, flushes any pending output and appends the current character data to the last character.
    - Updates the width of the last character if necessary and sets the new cell in the grid.
    - Redraws the combined cell and resets the cursor position.
- **Output**:
    - Returns 1 if the character was successfully combined, or 0 if it could not be combined.
- **Functions called**:
    - [`utf8_is_zwj`](utf8-combined.c.driver.md#utf8_is_zwj)
    - [`utf8_is_vs`](utf8-combined.c.driver.md#utf8_is_vs)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`grid_view_get_cell`](grid-view.c.driver.md#grid_view_get_cell)
    - [`utf8_is_modifier`](utf8-combined.c.driver.md#utf8_is_modifier)
    - [`utf8_has_zwj`](utf8-combined.c.driver.md#utf8_has_zwj)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`grid_view_set_cell`](grid-view.c.driver.md#grid_view_set_cell)
    - [`grid_view_set_padding`](grid-view.c.driver.md#grid_view_set_padding)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`tty_write`](tty.c.driver.md#tty_write)


---
### screen_write_overwrite<!-- {{#callable:screen_write_overwrite}} -->
The `screen_write_overwrite` function overwrites grid cells in a terminal screen context, handling padding and UTF-8 character widths.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `gc`: A pointer to a `grid_cell` structure representing the cell to be overwritten.
    - `width`: An unsigned integer representing the width of the character or characters to be written.
- **Control Flow**:
    - Checks if the `gc` cell has the `GRID_FLAG_PADDING` flag set, indicating it is a padding cell.
    - If it is a padding cell, it clears any leading padding cells back to the character and overwrites the current cell.
    - If the width is greater than 1 or the `gc` data width is greater than 1, it continues to overwrite any subsequent padding cells.
    - For each padding cell found, it either sets it to a default cell or modifies it if it is a tab character.
- **Output**:
    - Returns an integer indicating whether any cells were overwritten (1 if cells were overwritten, 0 otherwise).
- **Functions called**:
    - [`grid_view_get_cell`](grid-view.c.driver.md#grid_view_get_cell)
    - [`grid_view_set_cell`](grid-view.c.driver.md#grid_view_set_cell)


---
### screen_write_setselection<!-- {{#callable:screen_write_setselection}} -->
Sets the selection in the screen write context.
- **Inputs**:
    - `ctx`: Pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `flags`: A string representing flags that modify the selection behavior.
    - `str`: A pointer to a buffer containing the string to be set as the selection.
    - `len`: An unsigned integer representing the length of the string to be set as the selection.
- **Control Flow**:
    - Initializes a `tty_ctx` structure using [`screen_write_initctx`](#screen_write_initctx) with the provided `ctx`.
    - Sets the `ptr` field of `ttyctx` to point to the string `str`.
    - Sets the `ptr2` field of `ttyctx` to point to the `flags` string.
    - Sets the `num` field of `ttyctx` to the length `len`.
    - Calls [`tty_write`](tty.c.driver.md#tty_write) with the command `tty_cmd_setselection` and the initialized `ttyctx`.
- **Output**:
    - The function does not return a value; it performs an action to set the selection in the terminal.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`tty_write`](tty.c.driver.md#tty_write)


---
### screen_write_rawstring<!-- {{#callable:screen_write_rawstring}} -->
Writes a raw string to the screen context.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `str`: A pointer to a `u_char` array containing the string to be written.
    - `len`: An unsigned integer representing the length of the string to be written.
    - `allow_invisible_panes`: An integer flag indicating whether writing to invisible panes is allowed.
- **Control Flow**:
    - Initializes the `tty_ctx` structure using [`screen_write_initctx`](#screen_write_initctx) with the provided context.
    - Sets the `ptr` field of `ttyctx` to the string to be written and the `num` field to the length of the string.
    - Sets the `allow_invisible_panes` field of `ttyctx` to the provided flag.
    - Calls [`tty_write`](tty.c.driver.md#tty_write) with the command `tty_cmd_rawstring` and the initialized `ttyctx` to perform the actual writing.
- **Output**:
    - The function does not return a value; it performs a write operation to the screen based on the provided string and context.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`tty_write`](tty.c.driver.md#tty_write)


---
### screen_write_sixelimage<!-- {{#callable:screen_write_sixelimage}} -->
The `screen_write_sixelimage` function writes a SIXEL image to the screen, handling scaling and scrolling as necessary.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `si`: A pointer to a `sixel_image` structure representing the image to be written.
    - `bg`: An unsigned integer representing the background color to be used when writing the image.
- **Control Flow**:
    - The function first retrieves the dimensions of the SIXEL image using [`sixel_size_in_cells`](image-sixel.c.driver.md#sixel_size_in_cells).
    - It checks if the image dimensions exceed the screen size; if so, it scales the image using [`sixel_scale`](image-sixel.c.driver.md#sixel_scale).
    - If scaling fails, the function returns early.
    - Next, it calculates the available vertical space on the screen and determines if scrolling is needed.
    - If scrolling is required, it scrolls the screen up and updates the cursor position accordingly.
    - The function then flushes any collected screen write operations using [`screen_write_collect_flush`](#screen_write_collect_flush).
    - It initializes the TTY context and stores the image data using [`image_store`](image.c.driver.md#image_store).
    - Finally, it sends the command to write the SIXEL image to the TTY using [`tty_write`](tty.c.driver.md#tty_write) and moves the cursor to the appropriate position.
- **Output**:
    - The function does not return a value; it directly modifies the screen output by writing the SIXEL image and managing the cursor position.
- **Functions called**:
    - [`sixel_size_in_cells`](image-sixel.c.driver.md#sixel_size_in_cells)
    - [`sixel_scale`](image-sixel.c.driver.md#sixel_scale)
    - [`sixel_free`](image-sixel.c.driver.md#sixel_free)
    - [`image_scroll_up`](image.c.driver.md#image_scroll_up)
    - [`grid_view_scroll_region_up`](grid-view.c.driver.md#grid_view_scroll_region_up)
    - [`screen_write_collect_scroll`](#screen_write_collect_scroll)
    - [`screen_write_cursormove`](#screen_write_cursormove)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`image_store`](image.c.driver.md#image_store)
    - [`tty_write`](tty.c.driver.md#tty_write)


---
### screen_write_alternateon<!-- {{#callable:screen_write_alternateon}} -->
Enables the alternate screen mode for a terminal.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `gc`: A pointer to a `grid_cell` structure that defines the graphical attributes of the cell.
    - `cursor`: An integer indicating the cursor position or state.
- **Control Flow**:
    - Checks if the `window_pane` associated with the context is not NULL and if the alternate screen option is enabled.
    - If the alternate screen is not enabled, the function returns early without making any changes.
    - Flushes any collected screen write operations using [`screen_write_collect_flush`](#screen_write_collect_flush).
    - Calls [`screen_alternate_on`](screen.c.driver.md#screen_alternate_on) to activate the alternate screen mode.
    - If the `window_pane` is not NULL, it calls [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes) to adjust the layout of panes.
    - Initializes the `tty_ctx` structure for terminal context and checks if a redraw callback is set, invoking it if present.
- **Output**:
    - The function does not return a value but modifies the terminal's screen state to enable the alternate screen mode.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_alternate_on`](screen.c.driver.md#screen_alternate_on)
    - [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes)
    - [`screen_write_initctx`](#screen_write_initctx)


---
### screen_write_alternateoff<!-- {{#callable:screen_write_alternateoff}} -->
The `screen_write_alternateoff` function disables the alternate screen mode for a terminal context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `gc`: A pointer to a `grid_cell` structure that defines the graphical cell attributes.
    - `cursor`: An integer representing the cursor position or state.
- **Control Flow**:
    - The function first checks if the `window_pane` associated with the context is not NULL and if the alternate screen option is enabled; if not, it returns immediately.
    - It then calls [`screen_write_collect_flush`](#screen_write_collect_flush) to flush any collected screen write operations.
    - Next, it invokes [`screen_alternate_off`](screen.c.driver.md#screen_alternate_off) to disable the alternate screen mode.
    - If the `window_pane` is valid, it calls [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes) to adjust the layout of the panes in the window.
    - Finally, it initializes the TTY context and calls the redraw callback if it is set.
- **Output**:
    - The function does not return a value; it modifies the terminal's screen state by disabling the alternate screen mode and potentially triggers a redraw of the screen.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_alternate_off`](screen.c.driver.md#screen_alternate_off)
    - [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes)
    - [`screen_write_initctx`](#screen_write_initctx)


