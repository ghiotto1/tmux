# Purpose
This C source code file is part of the `tmux` terminal multiplexer and is responsible for handling screen writing operations. It provides a broad range of functionalities related to rendering text and managing the display within a terminal pane. The file includes functions for manipulating the screen buffer, such as inserting and deleting characters and lines, scrolling, and handling cursor movements. It also supports more complex operations like drawing boxes, menus, and handling UTF-8 character encoding, including combining characters and managing wide characters.

The code defines several static functions and structures that are used internally to manage screen state and rendering. It includes functions for setting up and tearing down screen writing contexts, managing cursor positions, and handling various screen modes. The file also contains functions for interacting with the terminal, such as setting the clipboard and writing raw strings. The presence of functions like [`screen_write_start`](#screen_write_start), [`screen_write_stop`](#screen_write_stop), and [`screen_write_cell`](#screen_write_cell) indicates that this file is a core component of the `tmux` rendering engine, providing essential APIs for screen manipulation. The code is designed to be integrated into the larger `tmux` application, and it does not define public APIs or external interfaces for use outside of this context.
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
- **Description**: The `screen_write_collect_trim` function is a static function that returns a pointer to a `screen_write_citem` structure. It is used within the context of screen writing operations, likely to manage or manipulate collected screen write items.
- **Use**: This function is used to trim collected screen write items based on specified parameters, aiding in efficient screen rendering.


---
### screen_write_citem_freelist
- **Type**: `TAILQ_HEAD(, screen_write_citem)`
- **Description**: The `screen_write_citem_freelist` is a global variable that represents a tail queue head for managing a list of `screen_write_citem` structures. It is initialized using the `TAILQ_HEAD_INITIALIZER` macro, which sets up the queue to be empty initially.
- **Use**: This variable is used to manage a free list of `screen_write_citem` structures, allowing for efficient allocation and deallocation of these items during screen writing operations.


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
- **Description**: The `screen_write_citem` structure is used to represent an item in a screen write operation, typically within a terminal or text-based user interface. It holds information about the position, type, and appearance of the item, including its coordinates, whether it is part of a wrapped line, its type (text or clear), and its background color. The structure also includes a `grid_cell` to define the cell's attributes and a TAILQ entry for managing the item in a linked list.


---
### screen_write_cline
- **Type**: `struct`
- **Members**:
    - `data`: A pointer to a character array, likely used to store line data.
    - `items`: A tail queue head for managing a list of `screen_write_citem` structures.
- **Description**: The `screen_write_cline` structure is designed to represent a line of text in a screen writing context, with `data` holding the actual text content and `items` managing a list of `screen_write_citem` structures that describe individual items or segments within the line. This structure is part of a larger system for managing screen output, likely in a terminal or text-based user interface, where lines of text need to be manipulated and displayed efficiently.


# Functions

---
### screen_write_get_citem<!-- {{#callable:screen_write_get_citem}} -->
The `screen_write_get_citem` function retrieves a `screen_write_citem` structure from a free list or allocates a new one if the list is empty.
- **Inputs**:
    - None
- **Control Flow**:
    - The function first attempts to get the first item from the `screen_write_citem_freelist` using `TAILQ_FIRST`.
    - If an item (`ci`) is found (i.e., `ci` is not NULL), it is removed from the free list using `TAILQ_REMOVE`, and its memory is cleared with `memset`.
    - The cleared item is then returned to the caller.
    - If no item is found in the free list, a new `screen_write_citem` is allocated using `xcalloc` and returned.
- **Output**:
    - The function returns a pointer to a `screen_write_citem` structure, either from the free list or a newly allocated one.


---
### screen_write_free_citem<!-- {{#callable:screen_write_free_citem}} -->
The `screen_write_free_citem` function adds a `screen_write_citem` structure back to the free list for reuse.
- **Inputs**:
    - `ci`: A pointer to a `screen_write_citem` structure that is to be freed and added back to the free list.
- **Control Flow**:
    - The function uses `TAILQ_INSERT_TAIL` to insert the `ci` item at the end of the `screen_write_citem_freelist` queue.
    - No conditions or loops are present; the function performs a single operation.
- **Output**:
    - The function does not return a value; it modifies the state of the free list by adding the specified `screen_write_citem` to it.


---
### screen_write_offset_timer<!-- {{#callable:screen_write_offset_timer}} -->
The `screen_write_offset_timer` function updates the window offset for a given window.
- **Inputs**:
    - `fd`: An integer file descriptor, which is unused in this function.
    - `events`: A short integer representing event types, which is also unused.
    - `data`: A pointer to a `void` type that is expected to point to a `window` structure.
- **Control Flow**:
    - The function retrieves the `window` structure from the `data` pointer.
    - It then calls the `tty_update_window_offset` function, passing the retrieved `window` as an argument.
- **Output**:
    - The function does not return a value; it performs an update on the window's offset.


---
### screen_write_set_cursor<!-- {{#callable:screen_write_set_cursor}} -->
Sets the cursor position on the screen.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `cx`: An integer representing the x-coordinate (column) of the cursor position.
    - `cy`: An integer representing the y-coordinate (row) of the cursor position.
- **Control Flow**:
    - The function first checks if the new cursor position is the same as the current position; if so, it returns immediately.
    - If the x-coordinate (cx) is not -1, it checks if it exceeds the screen width and adjusts it if necessary, then updates the screen's current x-coordinate.
    - If the y-coordinate (cy) is not -1, it checks if it exceeds the screen height and adjusts it if necessary, then updates the screen's current y-coordinate.
    - If the `window_pane` pointer in the context is NULL, the function returns.
    - If the `window` associated with the `window_pane` has not initialized its offset timer, it sets the timer to call `screen_write_offset_timer` after a delay.
    - If the timer is not already pending, it adds the timer to the event loop.
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
    - The function does not return a value; it modifies the state of the `window_pane` by setting its redraw flag.


---
### screen_write_set_client_cb<!-- {{#callable:screen_write_set_client_cb}} -->
Sets up the TTY context for a client based on the current pane and session.
- **Inputs**:
    - `ttyctx`: A pointer to a `tty_ctx` structure that holds the context for the terminal I/O.
    - `c`: A pointer to a `client` structure representing the client that is interacting with the terminal.
- **Control Flow**:
    - Checks if invisible panes are allowed; if so, verifies if the client's session has access to the current window pane.
    - If the current window is not the client's active window, or if the pane layout is null, the function returns 0.
    - Checks if the pane needs to be redrawn or dropped, returning -1 if so.
    - If the client has a redraw pending, it marks the pane for a deferred redraw and returns -1.
    - Calculates the window offset and updates the offsets in the `ttyctx` structure.
    - Adjusts the vertical offset in `ttyctx` if the status line is not active.
- **Output**:
    - Returns 1 if the setup is successful, 0 if the client cannot access the pane, or -1 if a redraw is required.


---
### screen_write_initctx<!-- {{#callable:screen_write_initctx}} -->
Initializes the `tty_ctx` structure for screen writing.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `ttyctx`: A pointer to the `tty_ctx` structure that will be initialized with screen dimensions and other properties.
    - `sync`: An integer flag indicating whether to synchronize updates.
- **Control Flow**:
    - The function starts by zeroing out the `ttyctx` structure.
    - It assigns the screen pointer and its dimensions to the `ttyctx` structure.
    - It copies default cell properties from `grid_default_cell` to `ttyctx`.
    - If a callback function is provided in `ctx`, it is invoked with `ctx` and `ttyctx` as arguments.
    - If the callback sets a palette, the foreground and background colors are updated if they are set to default values.
    - If no callback is provided, default redraw and client callback functions are set based on the context.
    - If the `SCREEN_WRITE_SYNC` flag is not set, it determines the synchronization behavior based on the active window pane and sends a synchronization command.
    - Finally, it updates the `ctx` flags to indicate that synchronization is now enabled.
- **Output**:
    - The function does not return a value but modifies the `ttyctx` structure to prepare it for screen writing operations.


---
### screen_write_make_list<!-- {{#callable:screen_write_make_list}} -->
The `screen_write_make_list` function initializes a write list for a given screen structure.
- **Inputs**:
    - `s`: A pointer to a `struct screen` that represents the screen for which the write list is being created.
- **Control Flow**:
    - Allocates memory for the `write_list` array in the `screen` structure using `xcalloc`, with the size determined by the screen's height.
    - Iterates over each row of the screen (from 0 to the height of the screen) and initializes a doubly linked list (`items`) for each entry in the `write_list` using `TAILQ_INIT`.
- **Output**:
    - The function does not return a value; it modifies the `write_list` field of the `screen` structure to prepare it for subsequent writing operations.


---
### screen_write_free_list<!-- {{#callable:screen_write_free_list}} -->
The `screen_write_free_list` function frees the memory allocated for the write list of a `screen` structure.
- **Inputs**:
    - `s`: A pointer to a `screen` structure whose write list is to be freed.
- **Control Flow**:
    - Iterates over each row of the screen's write list using a loop that runs from 0 to the height of the screen.
    - For each row, it frees the memory allocated for the `data` field of the `write_list` entry.
    - After freeing all row data, it frees the `write_list` itself.
- **Output**:
    - The function does not return a value; it performs memory deallocation to prevent memory leaks.


---
### screen_write_init<!-- {{#callable:screen_write_init}} -->
Initializes the `screen_write_ctx` structure for screen writing.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `s`: A pointer to the `screen` structure that represents the screen to be written to.
- **Control Flow**:
    - The function starts by zeroing out the `ctx` structure using `memset`.
    - It assigns the provided screen pointer `s` to the `ctx->s` field.
    - If the `write_list` of the screen is `NULL`, it calls [`screen_write_make_list`](#screen_write_make_list) to initialize it.
    - It retrieves a new `screen_write_citem` using [`screen_write_get_citem`](#screen_write_get_citem) and assigns it to `ctx->item`.
    - The `scrolled` field of the context is initialized to 0, and the background color is set to 8.
- **Output**:
    - The function does not return a value; it initializes the context for subsequent screen writing operations.
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
    - The [`screen_write_init`](#screen_write_init) function is called to initialize the writing context with the screen.
    - The window pane `wp` is assigned to the context `ctx`.
    - If the logging level is not zero, a debug log is generated that includes the size of the screen and the position of the pane.
- **Output**:
    - The function does not return a value; it modifies the `screen_write_ctx` to prepare it for writing to the specified pane.
- **Functions called**:
    - [`screen_write_init`](#screen_write_init)


---
### screen_write_start_callback<!-- {{#callable:screen_write_start_callback}} -->
Initializes the screen writing context with a callback function.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `s`: A pointer to the `screen` structure that represents the screen to be written to.
    - `cb`: A callback function of type `screen_write_init_ctx_cb` that will be called during initialization.
    - `arg`: A pointer to user-defined data that will be passed to the callback function.
- **Control Flow**:
    - Calls [`screen_write_init`](#screen_write_init) to initialize the context with the provided screen.
    - Sets the `init_ctx_cb` member of the context to the provided callback function.
    - Sets the `arg` member of the context to the provided argument.
    - If the logging level is greater than 0, logs the size of the screen along with the callback information.
- **Output**:
    - The function does not return a value; it modifies the `screen_write_ctx` structure and may log information based on the logging level.
- **Functions called**:
    - [`screen_write_init`](#screen_write_init)


---
### screen_write_start<!-- {{#callable:screen_write_start}} -->
Initializes the screen writing context for output.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `s`: A pointer to a `screen` structure that represents the screen to be written to.
- **Control Flow**:
    - Calls [`screen_write_init`](#screen_write_init) to initialize the context with the provided screen.
    - Checks the logging level; if it is not zero, logs the size of the screen and indicates that no pane is being used.
- **Output**:
    - The function does not return a value; it initializes the context for screen writing and may log debug information.
- **Functions called**:
    - [`screen_write_init`](#screen_write_init)


---
### screen_write_stop<!-- {{#callable:screen_write_stop}} -->
The `screen_write_stop` function finalizes the screen writing process by collecting and flushing any remaining output and freeing the current item.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing operations.
- **Control Flow**:
    - Calls `screen_write_collect_end(ctx)` to finalize the collection of screen write operations.
    - Calls `screen_write_collect_flush(ctx, 0, __func__)` to flush any remaining collected items to the screen.
    - Calls `screen_write_free_citem(ctx->item)` to free the current item in the context.
- **Output**:
    - The function does not return a value; it performs operations that affect the screen output and memory management.
- **Functions called**:
    - [`screen_write_collect_end`](#screen_write_collect_end)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_write_free_citem`](#screen_write_free_citem)


---
### screen_write_reset<!-- {{#callable:screen_write_reset}} -->
Resets the screen write context to its initial state.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
- **Control Flow**:
    - Calls `screen_reset_tabs` to reset tab stops on the screen.
    - Sets the scroll region of the screen to cover the entire height using [`screen_write_scrollregion`](#screen_write_scrollregion).
    - Initializes the screen mode to `MODE_CURSOR` and `MODE_WRAP`.
    - Checks if the global option for extended keys is set to 2, and if so, modifies the screen mode to include extended key modes.
    - Clears the screen using [`screen_write_clearscreen`](#screen_write_clearscreen) with a background color of 8.
    - Sets the cursor position to the top-left corner of the screen using [`screen_write_set_cursor`](#screen_write_set_cursor).
- **Output**:
    - The function does not return a value; it modifies the state of the screen and the context directly.
- **Functions called**:
    - [`screen_write_scrollregion`](#screen_write_scrollregion)
    - [`screen_write_clearscreen`](#screen_write_clearscreen)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_putc<!-- {{#callable:screen_write_putc}} -->
The `screen_write_putc` function writes a single character to the screen using a specified grid cell context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `gcp`: A pointer to a `grid_cell` structure that defines the attributes of the cell to be written.
    - `ch`: The character (of type `u_char`) to be written to the screen.
- **Control Flow**:
    - A local variable `gc` of type `grid_cell` is created to hold the cell data.
    - The `memcpy` function is used to copy the contents of `gcp` into `gc`.
    - The `utf8_set` function is called to set the character data in `gc` to the provided character `ch`.
    - The [`screen_write_cell`](#screen_write_cell) function is called with the context `ctx` and the modified cell `gc` to write the character to the screen.
- **Output**:
    - The function does not return a value; it directly modifies the screen by writing the specified character with the attributes defined in the `grid_cell`.
- **Functions called**:
    - [`screen_write_cell`](#screen_write_cell)


---
### screen_write_strlen<!-- {{#callable:screen_write_strlen}} -->
Calculates the display length of a formatted string considering UTF-8 encoding.
- **Inputs**:
    - `fmt`: A format string that specifies how to format the subsequent arguments.
- **Control Flow**:
    - Initializes a variable argument list using `va_start` to process additional arguments.
    - Allocates a formatted string using `xvasprintf` based on the format string and arguments.
    - Iterates through each character of the formatted string to calculate its display length.
    - Handles UTF-8 characters by checking if they are multi-byte and adjusting the length accordingly.
    - Counts regular characters and tabs, incrementing the size for each valid character.
    - Cleans up by freeing the allocated string before returning the calculated size.
- **Output**:
    - Returns the total display length of the formatted string as a `size_t` value.


---
### screen_write_text<!-- {{#callable:screen_write_text}} -->
The `screen_write_text` function writes formatted text to a specified area of the screen, handling line wrapping and cursor movement.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `cx`: An unsigned integer representing the starting x-coordinate (column) for writing text.
    - `width`: An unsigned integer specifying the width of the area in which to write the text.
    - `lines`: An unsigned integer indicating the number of lines available for writing.
    - `more`: An integer flag indicating whether more text will follow (1 for yes, 0 for no).
    - `gcp`: A pointer to a `grid_cell` structure that defines the attributes (like color) of the text to be written.
    - `fmt`: A format string for the text to be written, followed by a variable number of arguments.
- **Control Flow**:
    - The function starts by initializing a `va_list` to handle variable arguments and formats the input string using `xvasprintf`.
    - It converts the formatted string into UTF-8 data using `utf8_fromcstr`.
    - A loop is initiated to write text line by line until the specified number of lines is reached or the text is fully consumed.
    - Within the loop, it calculates how much text can fit on the current line, checking for line breaks and spaces.
    - The text is printed to the screen using [`screen_write_cell`](#screen_write_cell) for each character in the determined range.
    - If the end of the specified lines is reached or if there is no more text to write, the loop breaks.
    - Finally, it checks if there is more text to come and adjusts the cursor position accordingly before returning.
- **Output**:
    - The function returns 1 if the text was successfully written and more text is expected, or 0 if it reached the end of the specified lines or if there was more text to write but the area was full.
- **Functions called**:
    - [`screen_write_cell`](#screen_write_cell)
    - [`screen_write_cursormove`](#screen_write_cursormove)


---
### screen_write_puts<!-- {{#callable:screen_write_puts}} -->
The `screen_write_puts` function writes a formatted string to the screen using a specified grid cell context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `gcp`: A pointer to a `grid_cell` structure that defines the appearance of the text to be written.
    - `fmt`: A format string that specifies how subsequent arguments are converted for output.
- **Control Flow**:
    - The function starts by initializing a variable argument list using `va_start` with the format string `fmt`.
    - It then calls the [`screen_write_vnputs`](#screen_write_vnputs) function, passing the context, a maximum length of -1 (indicating no limit), the grid cell pointer, the format string, and the argument list.
    - Finally, it ends the variable argument list with `va_end`.
- **Output**:
    - The function does not return a value; it outputs formatted text to the screen based on the provided context and grid cell.
- **Functions called**:
    - [`screen_write_vnputs`](#screen_write_vnputs)


---
### screen_write_nputs<!-- {{#callable:screen_write_nputs}} -->
The `screen_write_nputs` function writes a formatted string to the screen with an optional maximum length.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `maxlen`: A `ssize_t` value that specifies the maximum number of characters to write; if -1, there is no limit.
    - `gcp`: A pointer to a `grid_cell` structure that defines the cell attributes for the text being written.
    - `fmt`: A format string that specifies how to format the output.
    - `...`: A variable number of additional arguments that are used to format the output according to the `fmt` string.
- **Control Flow**:
    - The function begins by initializing a variable argument list using `va_start` with the provided format string `fmt`.
    - It then calls the [`screen_write_vnputs`](#screen_write_vnputs) function, passing the context, maximum length, grid cell pointer, format string, and the argument list.
    - Finally, it cleans up the variable argument list using `va_end`.
- **Output**:
    - The function does not return a value; it writes formatted output to the screen based on the provided context and parameters.
- **Functions called**:
    - [`screen_write_vnputs`](#screen_write_vnputs)


---
### screen_write_vnputs<!-- {{#callable:screen_write_vnputs}} -->
The `screen_write_vnputs` function writes formatted output to a screen context, handling UTF-8 characters and various control characters.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `maxlen`: A `ssize_t` value that specifies the maximum number of characters to write; -1 indicates no limit.
    - `gcp`: A pointer to a `grid_cell` structure that defines the attributes of the characters to be written.
    - `fmt`: A format string that specifies how to format the output.
    - `ap`: A `va_list` that contains the variable arguments for the format string.
- **Control Flow**:
    - The function begins by copying the attributes from the provided `grid_cell` to a local variable.
    - It then formats the input string using `xvasprintf`, storing the result in `msg`.
    - A loop iterates over each character in the formatted string until the null terminator is reached.
    - If a character is a multi-byte UTF-8 character, it processes it accordingly, checking for width and appending it to the output.
    - If the character is a control character (like newline or tab), it handles it by calling appropriate functions to manage screen output.
    - The loop continues until either the end of the string is reached or the maximum length is exceeded.
    - Finally, the allocated memory for the formatted string is freed.
- **Output**:
    - The function does not return a value; instead, it writes characters to the screen context, potentially modifying the screen's state based on the input.
- **Functions called**:
    - [`screen_write_putc`](#screen_write_putc)
    - [`screen_write_cell`](#screen_write_cell)
    - [`screen_write_linefeed`](#screen_write_linefeed)
    - [`screen_write_carriagereturn`](#screen_write_carriagereturn)


---
### screen_write_fast_copy<!-- {{#callable:screen_write_fast_copy}} -->
Copies a rectangular region of cells from one `screen` to another.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `src`: A pointer to the source `screen` from which cells will be copied.
    - `px`: The x-coordinate of the top-left corner of the rectangle in the source screen.
    - `py`: The y-coordinate of the top-left corner of the rectangle in the source screen.
    - `nx`: The width of the rectangle to copy.
    - `ny`: The height of the rectangle to copy.
- **Control Flow**:
    - Checks if the width (nx) or height (ny) of the rectangle is zero; if so, it returns immediately.
    - Sets the cursor position to the current position of the destination screen.
    - Iterates over each row (yy) from py to py + ny in the source screen.
    - For each row, checks if the row index exceeds the height of the grid; if so, it breaks the loop.
    - Sets the cursor x-coordinate to the starting x-coordinate of the destination screen.
    - If a window pane is associated, initializes the TTY context.
    - Iterates over each column (xx) from px to px + nx in the current row.
    - Checks if the column index exceeds the line's cell size; if so, it breaks the loop.
    - Retrieves the cell at the current position from the source grid and checks if it fits within the specified width.
    - Sets the cell in the destination grid and writes it to the TTY if a window pane is associated.
    - Increments the cursor x-coordinate after each cell is written.
    - After processing all columns in a row, increments the cursor y-coordinate.
- **Output**:
    - The function does not return a value; it modifies the destination screen directly by copying the specified region from the source screen.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)


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
    - For each case corresponding to different box line styles (e.g., `BOX_LINES_DOUBLE`, `BOX_LINES_HEAVY`, etc.), the function clears the `GRID_ATTR_CHARSET` attribute from the `gc` structure and sets the `gc->data` to the appropriate border character using the `utf8_copy` or `utf8_set` functions.
    - In the cases of `BOX_LINES_SINGLE` and `BOX_LINES_DEFAULT`, the `GRID_ATTR_CHARSET` attribute is set instead.
- **Output**:
    - The function modifies the `grid_cell` structure pointed to by `gc` to represent the specified box border style, but does not return a value.


---
### screen_write_hline<!-- {{#callable:screen_write_hline}} -->
Draws a horizontal line on the screen.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `nx`: An unsigned integer representing the number of cells the horizontal line should span.
    - `left`: An integer flag indicating whether to draw a left join character at the start of the line.
    - `right`: An integer flag indicating whether to draw a right join character at the end of the line.
    - `lines`: An enumeration value of type `box_lines` that specifies the style of the line.
    - `border_gc`: A pointer to a `grid_cell` structure that defines the graphical properties of the border; if NULL, default properties are used.
- **Control Flow**:
    - Retrieve the current cursor position from the `screen` structure.
    - Check if `border_gc` is provided; if so, copy its properties to a local `grid_cell` variable, otherwise use default properties.
    - Set the character set attribute for the `grid_cell` used for drawing.
    - Determine the left border style based on the `left` flag and draw the first cell.
    - Draw the middle cells of the line based on the value of `nx`.
    - Determine the right border style based on the `right` flag and draw the last cell.
    - Restore the cursor position to its original location.
- **Output**:
    - The function does not return a value; it directly modifies the screen output by drawing a horizontal line with specified styles and attributes.
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
    - Copy the default grid cell attributes into a new grid cell structure (gc).
    - Set the character for the top of the line based on the 'top' flag and write it to the screen.
    - Iterate from 1 to ny-1 to draw the vertical line by setting the cursor position and writing the middle character ('x') for each line.
    - Set the cursor to the bottom position and write the bottom character based on the 'bottom' flag.
    - Reset the cursor position back to the original position (cx, cy).
- **Output**:
    - The function does not return a value; it directly modifies the screen output by drawing a vertical line composed of specified characters.
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
    - `lines`: An enumeration value indicating the type of box lines to be used for the menu border.
    - `menu_gc`: A pointer to a `grid_cell` structure that defines the appearance of the menu items.
    - `border_gc`: A pointer to a `grid_cell` structure that defines the appearance of the menu border.
    - `choice_gc`: A pointer to a `grid_cell` structure that defines the appearance of the selected menu item.
- **Control Flow**:
    - The function retrieves the current cursor position from the `screen` structure.
    - It initializes a default grid cell for menu items using the provided `menu_gc`.
    - A box is drawn around the menu using the [`screen_write_box`](#screen_write_box) function, which includes the title of the menu.
    - A loop iterates over each item in the menu, checking if the item name is NULL or if it matches the selected choice.
    - For each menu item, the cursor is moved to the appropriate position, and the item is drawn using the `format_draw` function.
    - If the item is a separator (indicated by a name starting with '-'), it is drawn in a dimmed style.
    - Finally, the cursor is reset to its original position after drawing the menu.
- **Output**:
    - The function does not return a value; it directly modifies the screen output to display the menu.
- **Functions called**:
    - [`screen_write_box`](#screen_write_box)
    - [`screen_write_cursormove`](#screen_write_cursormove)
    - [`screen_write_hline`](#screen_write_hline)
    - [`screen_write_putc`](#screen_write_putc)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_box<!-- {{#callable:screen_write_box}} -->
The `screen_write_box` function draws a rectangular box on the screen with specified borders and an optional title.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `nx`: An unsigned integer representing the width of the box.
    - `ny`: An unsigned integer representing the height of the box.
    - `lines`: An enumeration value of type `box_lines` that specifies the style of the box borders.
    - `gcp`: A pointer to a `grid_cell` structure that defines the cell attributes for the box; if NULL, default attributes are used.
    - `title`: A string representing the title to be displayed at the top of the box; if NULL, no title is displayed.
- **Control Flow**:
    - The function retrieves the current cursor position from the `screen` structure contained in `ctx`.
    - If `gcp` is not NULL, it copies its contents to a local `grid_cell` variable; otherwise, it initializes the variable with default cell attributes.
    - The function sets the character set attribute and disables the palette for the `grid_cell` used for drawing.
    - It draws the top border of the box by setting the appropriate border characters and writing them to the screen.
    - It draws the bottom border similarly after moving the cursor to the appropriate position.
    - The sides of the box are drawn by iterating through the height of the box and writing the side border characters.
    - If a title is provided, it formats and draws the title at the top of the box, adjusting the cell attributes accordingly.
    - Finally, the cursor is reset to its original position.
- **Output**:
    - The function does not return a value; it directly modifies the screen output by drawing the box with specified attributes and title.
- **Functions called**:
    - [`screen_write_box_border_set`](#screen_write_box_border_set)
    - [`screen_write_cell`](#screen_write_cell)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)
    - [`screen_write_cursormove`](#screen_write_cursormove)


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
    - It checks if the source screen is in cursor mode; if so, it calculates the preview area based on the cursor's position.
    - If the cursor is not in the preview area, it defaults to the top-left corner of the source screen.
    - The function then calls [`screen_write_fast_copy`](#screen_write_fast_copy) to copy the specified area from the source screen to the target context.
    - If the source screen is in cursor mode, it retrieves the cell at the cursor position, modifies its attributes to indicate it is selected, and writes it to the target context.
- **Output**:
    - The function does not return a value; it directly modifies the target screen context by writing the previewed area from the source screen.
- **Functions called**:
    - [`screen_write_fast_copy`](#screen_write_fast_copy)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)
    - [`screen_write_cell`](#screen_write_cell)


---
### screen_write_mode_set<!-- {{#callable:screen_write_mode_set}} -->
Sets the screen write mode for the given context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `mode`: An integer representing the mode to be set for the screen.
- **Control Flow**:
    - The function retrieves the `screen` structure from the `ctx` argument.
    - The specified `mode` is combined with the current mode using a bitwise OR operation.
    - If the logging level is not zero, a debug log is generated to indicate the mode change.
- **Output**:
    - The function does not return a value; it modifies the mode of the screen directly.


---
### screen_write_mode_clear<!-- {{#callable:screen_write_mode_clear}} -->
Clears a specified mode from the screen context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `mode`: An integer representing the mode to be cleared from the screen.
- **Control Flow**:
    - Retrieve the `screen` structure from the `ctx` context.
    - Clear the specified `mode` from the `screen`'s mode using a bitwise AND operation.
    - Check if the logging level is not zero.
    - If logging is enabled, log a debug message indicating the mode that was cleared.
- **Output**:
    - The function does not return a value; it modifies the state of the `screen` by clearing the specified mode.


---
### screen_write_cursorup<!-- {{#callable:screen_write_cursorup}} -->
Moves the cursor up by a specified number of lines.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
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
    - The function does not return a value; it directly modifies the cursor position on the screen.
- **Functions called**:
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_cursordown<!-- {{#callable:screen_write_cursordown}} -->
Moves the cursor down by a specified number of lines on the screen.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `ny`: An unsigned integer representing the number of lines to move the cursor down.
- **Control Flow**:
    - If `ny` is zero, it is set to one to ensure at least one line is moved.
    - The function checks if the current cursor position (`cy`) is above or below the defined scroll region (`rlower`).
    - If the cursor is below the `rlower`, it limits `ny` to the maximum number of lines that can be moved down without exceeding the screen size.
    - If the cursor is above the `rlower`, it limits `ny` to the number of lines remaining in the scroll region.
    - If the cursor is at the rightmost column, it decrements the column index (`cx`) to avoid overflow.
    - The cursor's vertical position (`cy`) is then incremented by `ny`.
    - Finally, the function calls [`screen_write_set_cursor`](#screen_write_set_cursor) to update the cursor's position.
- **Output**:
    - The function does not return a value; it directly modifies the cursor position on the screen.
- **Functions called**:
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_cursorright<!-- {{#callable:screen_write_cursorright}} -->
Moves the cursor to the right by a specified number of positions.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `nx`: An unsigned integer representing the number of positions to move the cursor to the right.
- **Control Flow**:
    - Checks if `nx` is zero and sets it to one if true, ensuring at least one position is moved.
    - Calculates the maximum number of positions the cursor can move based on the screen width and current cursor position.
    - If the calculated `nx` is greater than the available space, it adjusts `nx` to fit within the screen bounds.
    - If `nx` is still zero after adjustments, the function returns without making any changes.
    - Increments the current cursor x-position (`cx`) by `nx`.
    - Calls [`screen_write_set_cursor`](#screen_write_set_cursor) to update the cursor position in the context.
- **Output**:
    - The function does not return a value; it modifies the cursor position in the screen context.
- **Functions called**:
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_cursorleft<!-- {{#callable:screen_write_cursorleft}} -->
Moves the cursor left by a specified number of positions.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the current screen context.
    - `nx`: An unsigned integer representing the number of positions to move the cursor left.
- **Control Flow**:
    - If `nx` is zero, it is set to one to ensure at least one position is moved.
    - If `nx` exceeds the current cursor position (`cx`), it is adjusted to be equal to `cx`.
    - If `nx` is still zero after adjustments, the function returns without making any changes.
    - The cursor's x-coordinate (`cx`) is decremented by `nx`.
    - The [`screen_write_set_cursor`](#screen_write_set_cursor) function is called to update the cursor's position.
- **Output**:
    - The function does not return a value; it modifies the cursor position on the screen directly.
- **Functions called**:
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_backspace<!-- {{#callable:screen_write_backspace}} -->
The `screen_write_backspace` function moves the cursor left by one position, handling line wrapping appropriately.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing operations.
- **Control Flow**:
    - Check if the current cursor x-coordinate (`cx`) is 0.
    - If `cx` is 0 and the y-coordinate (`cy`) is also 0, return immediately as there is no space to move.
    - If `cx` is 0 but `cy` is greater than 0, retrieve the previous line from the grid.
    - Check if the previous line is wrapped; if so, decrement `cy` and set `cx` to the last column of the screen.
    - If `cx` is not 0, simply decrement `cx` by 1.
    - Finally, call [`screen_write_set_cursor`](#screen_write_set_cursor) to update the cursor position.
- **Output**:
    - The function does not return a value but updates the cursor position in the screen context.
- **Functions called**:
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_alignmenttest<!-- {{#callable:screen_write_alignmenttest}} -->
The `screen_write_alignmenttest` function performs a VT100 alignment test by filling the screen with a specific character and resetting the cursor.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
- **Control Flow**:
    - The function begins by copying the default grid cell into a local variable `gc` and setting its data to the character 'E'.
    - If the `ENABLE_SIXEL` macro is defined, it checks if all images can be freed and sets the redraw flag for the window pane if applicable.
    - It then iterates over each cell in the screen's grid, setting each cell to the character 'E'.
    - After filling the grid, it resets the cursor position to the top-left corner of the screen.
    - The function sets the screen's upper and lower scroll regions.
    - It initializes the TTY context for writing and collects commands to clear the screen before sending the alignment test command.
- **Output**:
    - The function does not return a value but modifies the screen's grid and sends a command to the TTY to perform an alignment test.
- **Functions called**:
    - [`screen_write_set_cursor`](#screen_write_set_cursor)
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`screen_write_collect_clear`](#screen_write_collect_clear)


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
    - The collected changes are flushed to the screen, and the insertion command is sent to the terminal.
- **Output**:
    - The function does not return a value; it modifies the screen's grid and sends commands to the terminal to reflect the inserted characters.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)


---
### screen_write_deletecharacter<!-- {{#callable:screen_write_deletecharacter}} -->
Deletes a specified number of characters from the current cursor position in a screen context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `nx`: An unsigned integer representing the number of characters to delete.
    - `bg`: An unsigned integer representing the background color to use for the deleted characters.
- **Control Flow**:
    - If `nx` is zero, it is set to one to ensure at least one character is deleted.
    - If `nx` exceeds the number of characters available from the current cursor position to the end of the line, it is adjusted to that maximum.
    - If `nx` is still zero after adjustments, the function returns immediately.
    - If the cursor's x-coordinate is out of bounds, the function returns immediately.
    - If the `ENABLE_SIXEL` feature is enabled and the current line has an image, the pane is marked for redraw.
    - The screen writing context is initialized for the operation.
    - The `grid_view_delete_cells` function is called to perform the actual deletion of characters from the grid.
    - The collected changes are flushed to the output, and a TTY command to delete characters is sent.
- **Output**:
    - The function does not return a value; it modifies the screen's grid and sends commands to the TTY to reflect the deletion of characters.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)


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
    - The collected screen write operations are flushed to ensure the changes are applied.
    - The terminal is instructed to clear the specified number of characters using the appropriate command.
- **Output**:
    - The function does not return a value; it performs operations to clear characters on the screen and updates the terminal accordingly.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)


---
### screen_write_insertline<!-- {{#callable:screen_write_insertline}} -->
Inserts a specified number of lines into the screen at the current cursor position.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `ny`: An unsigned integer representing the number of lines to insert.
    - `bg`: An unsigned integer representing the background color to use for the inserted lines.
- **Control Flow**:
    - If `ny` is zero, it is set to one to ensure at least one line is inserted.
    - If the current cursor position (`s->cy`) is outside the defined scroll region (`s->rupper` and `s->rlower`), the function checks if the number of lines to insert exceeds the available space and adjusts `ny` accordingly.
    - The function initializes the `ttyctx` structure for terminal writing and sets the background color.
    - It calls `grid_view_insert_lines` to perform the actual insertion of lines into the grid.
    - The function collects any pending screen write operations and flushes them to the terminal.
    - Finally, it sends a command to the terminal to insert the specified number of lines.
- **Output**:
    - The function does not return a value; it modifies the screen's grid and sends commands to the terminal to reflect the changes.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)


---
### screen_write_deleteline<!-- {{#callable:screen_write_deleteline}} -->
Deletes a specified number of lines from the screen.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `ny`: The number of lines to delete from the screen.
    - `bg`: The background color to use when deleting lines.
- **Control Flow**:
    - If `ny` is zero, it is set to one to ensure at least one line is deleted.
    - If the current cursor position is outside the defined scroll region, the function checks if the number of lines to delete exceeds the available space and adjusts accordingly.
    - The function initializes the TTY context and sets the background color.
    - It calls `grid_view_delete_lines` or `grid_view_delete_lines_region` to perform the actual deletion of lines from the grid.
    - Finally, it flushes the collected screen write commands and sends a TTY command to delete the specified number of lines.
- **Output**:
    - The function does not return a value; it modifies the screen by deleting the specified number of lines.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)


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
    - [`screen_write_collect_clear`](#screen_write_collect_clear)
    - [`screen_write_get_citem`](#screen_write_get_citem)


---
### screen_write_clearendofline<!-- {{#callable:screen_write_clearendofline}} -->
Clears the line from the cursor to the end of the line in the screen context.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `bg`: An unsigned integer representing the background color to be used for clearing.
- **Control Flow**:
    - Checks if the cursor is at the beginning of the line; if so, it calls [`screen_write_clearline`](#screen_write_clearline) to clear the entire line.
    - Retrieves the current line from the grid using the cursor's y-coordinate.
    - Validates if the cursor's x-coordinate is within the screen width and if it is not beyond the current line's cell size.
    - If the conditions are met, it clears the cells from the cursor position to the end of the line with the specified background color.
    - Collects the clear operation into the context's write list, adjusting the entry's properties accordingly.
- **Output**:
    - The function does not return a value but modifies the screen's grid by clearing the specified area and updates the context for future operations.
- **Functions called**:
    - [`screen_write_clearline`](#screen_write_clearline)
    - [`screen_write_collect_trim`](#screen_write_collect_trim)
    - [`screen_write_get_citem`](#screen_write_get_citem)


---
### screen_write_clearstartofline<!-- {{#callable:screen_write_clearstartofline}} -->
Clears the start of the current line in the screen context.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that contains the context for screen writing.
    - `bg`: An unsigned integer representing the background color to be used for clearing.
- **Control Flow**:
    - Checks if the current cursor position is at or beyond the right edge of the screen; if so, it calls [`screen_write_clearline`](#screen_write_clearline) to clear the entire line.
    - If the cursor position is valid, it checks if the cursor is beyond the screen width and clears the line accordingly.
    - It collects any existing items that need to be trimmed from the write list before inserting the new clear item.
    - The new clear item is then inserted into the write list at the appropriate position.
- **Output**:
    - The function does not return a value but modifies the screen context by clearing part of the line and updating the write list.
- **Functions called**:
    - [`screen_write_clearline`](#screen_write_clearline)
    - [`screen_write_collect_trim`](#screen_write_collect_trim)
    - [`screen_write_get_citem`](#screen_write_get_citem)


---
### screen_write_cursormove<!-- {{#callable:screen_write_cursormove}} -->
Moves the cursor to a specified position on the screen.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `px`: An integer representing the x-coordinate (column) to move the cursor to.
    - `py`: An integer representing the y-coordinate (row) to move the cursor to.
    - `origin`: An integer that indicates whether to adjust the y-coordinate based on the origin mode.
- **Control Flow**:
    - Checks if the `origin` flag is set and if `py` is not -1; if so, adjusts `py` based on the screen's origin mode.
    - Validates the `px` and `py` coordinates to ensure they do not exceed the screen boundaries.
    - Logs the current and new cursor positions for debugging purposes.
    - Calls the [`screen_write_set_cursor`](#screen_write_set_cursor) function to actually move the cursor to the new position.
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
    - Checks if the current cursor position (`s->cy`) is at the top of the scroll region (`s->rupper`).
    - If at the top, it calls `image_free_all` to potentially free images and marks the pane for redraw if necessary.
    - Calls `grid_view_scroll_region_down` to scroll the visible region down by one line.
    - Flushes any collected screen write operations using [`screen_write_collect_flush`](#screen_write_collect_flush).
    - Initializes the TTY context with [`screen_write_initctx`](#screen_write_initctx) and sets the background color.
    - Sends a command to the TTY to perform a reverse index operation using `tty_write`.
    - If the cursor is not at the top, it checks if the cursor is above the first line and moves it up by one line.
- **Output**:
    - The function does not return a value but modifies the screen's state by scrolling the content and potentially updating the cursor position.
- **Functions called**:
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_scrollregion<!-- {{#callable:screen_write_scrollregion}} -->
Sets the scroll region for the screen.
- **Inputs**:
    - `ctx`: Pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `rupper`: The upper boundary of the scroll region.
    - `rlower`: The lower boundary of the scroll region.
- **Control Flow**:
    - Checks if `rupper` exceeds the maximum screen height and adjusts it accordingly.
    - Checks if `rlower` exceeds the maximum screen height and adjusts it accordingly.
    - Validates that `rupper` is less than `rlower` to ensure a valid scroll region.
    - Flushes any collected screen write operations.
    - Sets the cursor position to the top-left corner of the screen.
    - Updates the screen's `rupper` and `rlower` properties to define the new scroll region.
- **Output**:
    - The function does not return a value; it modifies the state of the screen by setting the scroll region.
- **Functions called**:
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_linefeed<!-- {{#callable:screen_write_linefeed}} -->
The `screen_write_linefeed` function handles the line feed operation in a terminal screen context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `wrapped`: An integer indicating whether the line is wrapped.
    - `bg`: An unsigned integer representing the background color to be used.
- **Control Flow**:
    - The function retrieves the current screen and grid from the context.
    - If the `wrapped` parameter is true, it sets the corresponding flag in the current grid line.
    - It logs the current cursor position and the screen region.
    - If the background color has changed, it flushes the collected output and updates the background color in the context.
    - If the cursor is at the bottom of the screen, it checks if the screen needs to scroll up and performs the scrolling if necessary.
    - If the cursor is not at the bottom, it simply moves the cursor down to the next line.
- **Output**:
    - The function does not return a value; it modifies the screen context and updates the cursor position accordingly.
- **Functions called**:
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_write_collect_scroll`](#screen_write_collect_scroll)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_scrollup<!-- {{#callable:screen_write_scrollup}} -->
The `screen_write_scrollup` function scrolls the visible region of a terminal screen up by a specified number of lines.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing operations.
    - `lines`: An unsigned integer representing the number of lines to scroll up; if zero, defaults to one.
    - `bg`: An unsigned integer representing the background color to use during the scroll operation.
- **Control Flow**:
    - If `lines` is zero, it is set to one.
    - If `lines` exceeds the scrollable region, it is adjusted to the maximum scrollable lines.
    - If the background color `bg` differs from the current context's background, the context is flushed and updated.
    - If the `ENABLE_SIXEL` feature is enabled, it checks if an image scroll is needed and marks the pane for redraw if necessary.
    - A loop iterates over the number of lines to scroll, calling `grid_view_scroll_region_up` to perform the actual scrolling for each line.
    - After scrolling, [`screen_write_collect_scroll`](#screen_write_collect_scroll) is called to collect the scroll operation.
- **Output**:
    - The function does not return a value but modifies the screen's visible content by scrolling it up and updating the context accordingly.
- **Functions called**:
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_write_collect_scroll`](#screen_write_collect_scroll)


---
### screen_write_scrolldown<!-- {{#callable:screen_write_scrolldown}} -->
Scrolls the visible region of the screen down by a specified number of lines.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `lines`: An unsigned integer representing the number of lines to scroll down.
    - `bg`: An unsigned integer representing the background color to use during the scroll.
- **Control Flow**:
    - Initializes the TTY context with [`screen_write_initctx`](#screen_write_initctx) and sets the background color.
    - Checks if `lines` is zero and sets it to one if true.
    - Ensures `lines` does not exceed the scrollable region defined by `s->rlower` and `s->rupper`.
    - If the `ENABLE_SIXEL` flag is set, it checks for any free images and marks the pane for redraw if necessary.
    - Iterates over the number of lines to scroll down, calling `grid_view_scroll_region_down` for each line.
    - Flushes the collected screen write commands with [`screen_write_collect_flush`](#screen_write_collect_flush).
    - Sets the number of lines scrolled in the TTY context and sends the scroll command to the TTY.
- **Output**:
    - The function does not return a value but performs a scroll operation on the screen and updates the TTY with the new state.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)


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
    - Sends a command to the terminal to clear from the cursor to the end of the screen.
- **Output**:
    - No return value; the function performs operations to clear the screen and updates the terminal display accordingly.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`screen_write_collect_clear`](#screen_write_collect_clear)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)


---
### screen_write_clearstartofscreen<!-- {{#callable:screen_write_clearstartofscreen}} -->
Clears the screen from the start to the current cursor position.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `bg`: An unsigned integer representing the background color to be used for clearing.
- **Control Flow**:
    - The function retrieves the current screen from the context.
    - It checks if the `ENABLE_SIXEL` feature is enabled and if the image needs to be redrawn.
    - The function initializes the TTY context with the specified background color.
    - It clears the grid from the top of the screen to the current cursor position, handling both the current line and any lines above it.
    - It collects the clear operation for later processing.
    - Finally, it flushes the collected operations and sends a command to the TTY to clear the start of the screen.
- **Output**:
    - The function does not return a value; it performs operations to clear the screen and update the display accordingly.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`screen_write_collect_clear`](#screen_write_collect_clear)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)


---
### screen_write_clearscreen<!-- {{#callable:screen_write_clearscreen}} -->
Clears the entire screen and optionally scrolls the history.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `bg`: An unsigned integer representing the background color to be used when clearing the screen.
- **Control Flow**:
    - Checks if the `ENABLE_SIXEL` feature is enabled and if so, calls `image_free_all` to free any images and sets the redraw flag for the window pane if necessary.
    - Initializes the `tty_ctx` structure with the [`screen_write_initctx`](#screen_write_initctx) function, setting the background color.
    - Checks if the screen's grid has the `GRID_HISTORY` flag set and if the option `scroll-on-clear` is enabled; if so, it calls `grid_view_clear_history` to clear the history.
    - If the history is not cleared, it calls `grid_view_clear` to clear the entire grid area with the specified background color.
    - Calls [`screen_write_collect_clear`](#screen_write_collect_clear) to collect the clear operation for the current context.
    - Finally, sends the clear screen command to the terminal using `tty_write`.
- **Output**:
    - The function does not return a value; it performs operations to clear the screen and update the display accordingly.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`screen_write_collect_clear`](#screen_write_collect_clear)


---
### screen_write_clearhistory<!-- {{#callable:screen_write_clearhistory}} -->
Clears the entire history of the screen.
- **Inputs**:
    - None
- **Control Flow**:
    - The function directly calls `grid_clear_history` with the grid associated with the `screen_write_ctx` structure.
- **Output**:
    - The function does not return any value; it performs an action that clears the screen's history.


---
### screen_write_fullredraw<!-- {{#callable:screen_write_fullredraw}} -->
The `screen_write_fullredraw` function triggers a complete redraw of the screen context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
- **Control Flow**:
    - Calls [`screen_write_collect_flush`](#screen_write_collect_flush) to flush any collected screen write operations.
    - Initializes a `tty_ctx` structure using [`screen_write_initctx`](#screen_write_initctx).
    - If the `redraw_cb` callback in `ttyctx` is not NULL, it invokes this callback, passing the `ttyctx` as an argument.
- **Output**:
    - The function does not return a value but performs operations that result in the screen being fully redrawn based on the current context.
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
    - `wrapped`: A pointer to an integer that indicates if the trimming resulted in a wrapped line.
- **Control Flow**:
    - Checks if the list of items for the specified row is empty; if so, returns NULL.
    - Iterates through each item in the list using a safe loop to avoid modification during iteration.
    - For each item, calculates its start and end positions based on its `x` and `used` properties.
    - Compares the item's position with the specified trim coordinates to determine if it is before, after, inside, or overlapping the trim area.
    - If the item is entirely before the trim area, it continues to the next item.
    - If the item is entirely after the trim area, it sets the `before` pointer to the current item and breaks the loop.
    - If the item is entirely inside the trim area, it removes the item from the list and frees it, checking if it was wrapped.
    - If the item overlaps the start of the trim area, it adjusts the item's `used` property accordingly.
    - If the item overlaps the end of the trim area, it adjusts the item's `x` and `used` properties.
    - If the item covers both sides of the trim area, it splits the item into two, creating a new item for the right side.
- **Output**:
    - Returns a pointer to the item that is before the trimmed area, or NULL if no such item exists.
- **Functions called**:
    - [`screen_write_free_citem`](#screen_write_free_citem)
    - [`screen_write_get_citem`](#screen_write_get_citem)


---
### screen_write_collect_clear<!-- {{#callable:screen_write_collect_clear}} -->
The `screen_write_collect_clear` function clears collected lines from the screen write context.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `y`: An unsigned integer representing the starting line number from which to clear collected items.
    - `n`: An unsigned integer representing the number of lines to clear from the starting line.
- **Control Flow**:
    - The function iterates from the starting line `y` to `y + n`.
    - For each line, it retrieves the corresponding `screen_write_cline` structure from the context's write list.
    - It then concatenates the items of the current line to a free list, effectively clearing them.
- **Output**:
    - The function does not return a value; it modifies the internal state of the `screen_write_ctx` by clearing the collected items.


---
### screen_write_collect_scroll<!-- {{#callable:screen_write_collect_scroll}} -->
The `screen_write_collect_scroll` function scrolls the screen region up and collects the necessary screen write operations.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `bg`: An unsigned integer representing the background color to be used during the scroll operation.
- **Control Flow**:
    - Logs the current cursor position and the scroll region boundaries.
    - Calls [`screen_write_collect_clear`](#screen_write_collect_clear) to clear the current line at the upper scroll region.
    - Saves the data of the upper scroll region line.
    - Iterates from the upper scroll region to the lower scroll region, concatenating items from the next line into the current line.
    - Sets the data of the lower scroll region line to the saved data.
    - Creates a new `screen_write_citem` for the clear operation at the lower scroll region and inserts it into the write list.
- **Output**:
    - The function does not return a value but modifies the screen write context to reflect the new state after scrolling.
- **Functions called**:
    - [`screen_write_collect_clear`](#screen_write_collect_clear)
    - [`screen_write_get_citem`](#screen_write_get_citem)


---
### screen_write_collect_flush<!-- {{#callable:screen_write_collect_flush}} -->
The `screen_write_collect_flush` function processes and flushes collected screen write items to the terminal.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `scroll_only`: An integer flag indicating whether to only perform scrolling without flushing other items.
    - `from`: A string indicating the source or context from which the flush is being called.
- **Control Flow**:
    - Checks if there are any scrolled items in the context and logs the scrolling action.
    - If scrolling is required, it adjusts the scroll amount to fit within the screen's scrollable region.
    - Initializes the TTY context for scrolling and sends a scroll command to the terminal.
    - Resets the scrolled count and background color in the context.
    - If `scroll_only` is false, it proceeds to flush the collected items.
    - Iterates over each line in the screen's write list, processing each collected item.
    - For each item, it sets the cursor position and sends the appropriate command to the terminal based on the item type (CLEAR or TEXT).
    - Removes the processed item from the collection and frees its memory.
    - Finally, it restores the cursor position to its original state and logs the number of flushed items.
- **Output**:
    - The function does not return a value; instead, it sends commands to the terminal to update the display based on the collected screen write items.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)
    - [`screen_write_free_citem`](#screen_write_free_citem)


---
### screen_write_collect_end<!-- {{#callable:screen_write_collect_end}} -->
The `screen_write_collect_end` function finalizes the collection of screen write items and updates the screen accordingly.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing operations.
- **Control Flow**:
    - Checks if the `used` field of the current item is zero; if so, it returns immediately.
    - Calls [`screen_write_collect_trim`](#screen_write_collect_trim) to trim collected items and determine where to insert the current item.
    - Updates the `x` position and `wrapped` status of the current item based on the trimming results.
    - Inserts the current item into the appropriate position in the linked list of collected items.
    - Logs the details of the collected item for debugging purposes.
    - If the current cursor position is not at the start of the line, it clears any padding cells to the left of the cursor.
    - Checks for image areas that may need redrawing if the Sixel extension is enabled.
    - Sets the cells in the grid with the collected item data and updates the cursor position.
    - Clears any padding cells to the right of the cursor position.
- **Output**:
    - The function does not return a value but modifies the state of the screen by updating the grid with the collected item data and adjusting the cursor position.
- **Functions called**:
    - [`screen_write_collect_trim`](#screen_write_collect_trim)
    - [`screen_write_get_citem`](#screen_write_get_citem)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_collect_add<!-- {{#callable:screen_write_collect_add}} -->
The `screen_write_collect_add` function adds a character cell to the current screen context, collecting it if certain conditions are met.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the current screen context.
    - `gc`: A pointer to a `grid_cell` structure representing the character cell to be added.
- **Control Flow**:
    - The function first checks if the character cell can be collected based on its attributes and the current screen mode.
    - If the cell cannot be collected, it ends the current collection, flushes any collected data, and writes the cell directly to the screen.
    - If the cell can be collected, it checks if the current cursor position exceeds the screen width and ends the collection if necessary.
    - If the cursor position is valid, it updates the collected item with the new cell data and increments the used count.
- **Output**:
    - The function does not return a value; it modifies the screen context and potentially updates the display with the new character cell.
- **Functions called**:
    - [`screen_write_collect_end`](#screen_write_collect_end)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_write_cell`](#screen_write_cell)
    - [`screen_write_linefeed`](#screen_write_linefeed)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)


---
### screen_write_cell<!-- {{#callable:screen_write_cell}} -->
The `screen_write_cell` function writes a character cell to the screen, handling various conditions such as wrapping, combining characters, and selection.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `gc`: A pointer to a `grid_cell` structure that represents the cell data to be written to the screen.
- **Control Flow**:
    - Checks if the cell is a padding cell and returns early if true.
    - Attempts to combine the current cell with the previous one if applicable.
    - Flushes any existing scrolling before writing the new cell.
    - Checks if the character fits within the current line and handles wrapping if necessary.
    - In insert mode, it makes space for the new cells.
    - Checks the cursor position and ensures it is valid.
    - Handles overwriting of existing UTF-8 characters if necessary.
    - Fills in padding cells if the new character is wide.
    - Determines if the new cell is different from the existing one and updates the grid accordingly.
    - Updates the selection state of the cell if necessary.
    - Moves the cursor to the appropriate position after writing the cell.
    - Writes the cell data to the screen using the appropriate TTY command.
- **Output**:
    - The function does not return a value but updates the screen's grid with the new cell data and manages the cursor position accordingly.
- **Functions called**:
    - [`screen_write_combine`](#screen_write_combine)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_write_linefeed`](#screen_write_linefeed)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)
    - [`screen_write_initctx`](#screen_write_initctx)
    - [`screen_write_overwrite`](#screen_write_overwrite)


---
### screen_write_combine<!-- {{#callable:screen_write_combine}} -->
Combines a UTF-8 character with the previous character in the grid if applicable.
- **Inputs**:
    - `ctx`: A pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `gc`: A pointer to the `grid_cell` structure representing the character to be written.
- **Control Flow**:
    - Checks if the character is a zero-width joiner (ZWJ) or a variation selector (VS) and sets flags accordingly.
    - Returns early if the character is empty or if it is at the leftmost position in the grid.
    - Retrieves the last character in the grid to determine if combining is possible.
    - Checks if the last character can be combined with the current character based on width and padding.
    - If combining is valid, it appends the current character data to the last character's data.
    - Adjusts the width of the last character if necessary and updates the grid cell.
    - Flushes any pending output and redraws the combined character.
- **Output**:
    - Returns 1 if the character was successfully combined, or 0 if it could not be combined.
- **Functions called**:
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_write_set_cursor`](#screen_write_set_cursor)
    - [`screen_write_initctx`](#screen_write_initctx)


---
### screen_write_overwrite<!-- {{#callable:screen_write_overwrite}} -->
The `screen_write_overwrite` function overwrites grid cells in a terminal screen context, handling padding and UTF-8 character widths.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `gc`: A pointer to a `grid_cell` structure representing the cell to be overwritten.
    - `width`: An unsigned integer representing the width of the character to be written.
- **Control Flow**:
    - Checks if the `gc` cell has the `GRID_FLAG_PADDING` flag set.
    - If it is a padding cell, it clears any leading and trailing padding cells until a non-padding cell is found.
    - Logs the position of the padding and overwrites it with a default cell.
    - If the width is greater than 1 or the cell is not a single-width character, it continues to overwrite subsequent cells.
    - Logs the overwriting of each cell and sets them to either a default cell or modifies them if they are tabs.
- **Output**:
    - Returns an integer indicating whether any cells were overwritten (1 if cells were overwritten, 0 otherwise).


---
### screen_write_setselection<!-- {{#callable:screen_write_setselection}} -->
Sets the selection in the terminal screen context.
- **Inputs**:
    - `ctx`: Pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `flags`: A string representing flags that modify the selection behavior.
    - `str`: A pointer to the string that represents the selection content.
    - `len`: An unsigned integer representing the length of the selection string.
- **Control Flow**:
    - Initializes a `tty_ctx` structure using [`screen_write_initctx`](#screen_write_initctx) with the provided `ctx`.
    - Sets the `ptr` field of `ttyctx` to point to the selection string `str`.
    - Sets the `ptr2` field of `ttyctx` to point to the `flags` string.
    - Sets the `num` field of `ttyctx` to the length of the selection string `len`.
    - Calls `tty_write` with the command `tty_cmd_setselection` and the initialized `ttyctx` to perform the selection operation.
- **Output**:
    - The function does not return a value; it performs an operation to set the selection in the terminal.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)


---
### screen_write_rawstring<!-- {{#callable:screen_write_rawstring}} -->
Writes a raw string to the screen context.
- **Inputs**:
    - `ctx`: Pointer to the `screen_write_ctx` structure that holds the context for screen writing.
    - `str`: Pointer to the string (of type `u_char*`) that needs to be written to the screen.
    - `len`: An unsigned integer representing the length of the string to be written.
    - `allow_invisible_panes`: An integer flag indicating whether writing to invisible panes is allowed.
- **Control Flow**:
    - Initializes the `tty_ctx` structure using [`screen_write_initctx`](#screen_write_initctx) with the provided context.
    - Sets the `ptr` field of `ttyctx` to the input string `str`.
    - Sets the `num` field of `ttyctx` to the input length `len`.
    - Sets the `allow_invisible_panes` field of `ttyctx` to the input flag.
    - Calls `tty_write` with the command `tty_cmd_rawstring` and the initialized `ttyctx`.
- **Output**:
    - The function does not return a value; it performs a write operation to the screen based on the provided string and context.
- **Functions called**:
    - [`screen_write_initctx`](#screen_write_initctx)


---
### screen_write_sixelimage<!-- {{#callable:screen_write_sixelimage}} -->
The `screen_write_sixelimage` function writes a SIXEL image to the screen, handling scaling and scrolling as necessary.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that contains the context for screen writing.
    - `si`: A pointer to a `sixel_image` structure representing the image to be written.
    - `bg`: An unsigned integer representing the background color to be used when writing the image.
- **Control Flow**:
    - The function first retrieves the dimensions of the SIXEL image using `sixel_size_in_cells`.
    - It checks if the image dimensions exceed the screen size; if so, it scales the image using `sixel_scale`.
    - If the image cannot be scaled, the function exits early.
    - Next, it calculates the available vertical space on the screen and determines if scrolling is needed.
    - If scrolling is required, it scrolls the screen up and updates the cursor position accordingly.
    - The function then flushes any collected screen write operations.
    - It initializes the TTY context and stores the image data using `image_store`.
    - Finally, it sends the command to write the SIXEL image to the TTY.
- **Output**:
    - The function does not return a value; it directly modifies the screen output by writing the SIXEL image and managing the cursor position.
- **Functions called**:
    - [`screen_write_collect_scroll`](#screen_write_collect_scroll)
    - [`screen_write_cursormove`](#screen_write_cursormove)
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_write_initctx`](#screen_write_initctx)


---
### screen_write_alternateon<!-- {{#callable:screen_write_alternateon}} -->
Enables the alternate screen buffer for terminal display.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure that holds the context for screen writing.
    - `gc`: A pointer to a `grid_cell` structure that defines the graphical cell attributes.
    - `cursor`: An integer indicating the cursor position or state.
- **Control Flow**:
    - Checks if the `window_pane` associated with the context is not NULL and if the alternate screen option is enabled.
    - If the alternate screen is not enabled, the function returns early without making any changes.
    - Flushes any collected screen write operations using [`screen_write_collect_flush`](#screen_write_collect_flush).
    - Calls `screen_alternate_on` to activate the alternate screen buffer with the provided context, grid cell, and cursor state.
    - If the `window_pane` is not NULL, it calls `layout_fix_panes` to adjust the layout of the panes.
    - Initializes the `tty_ctx` structure for terminal context and checks if a redraw callback is set, invoking it if available.
- **Output**:
    - The function does not return a value; it modifies the terminal's display state to enable the alternate screen buffer.
- **Functions called**:
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
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
    - It then flushes any collected screen write operations using [`screen_write_collect_flush`](#screen_write_collect_flush).
    - The function calls `screen_alternate_off` to actually turn off the alternate screen mode.
    - If the `window_pane` is not NULL, it calls `layout_fix_panes` to adjust the layout of the panes in the window.
    - Finally, it initializes the TTY context and invokes the redraw callback if it is set.
- **Output**:
    - The function does not return a value; it modifies the terminal's screen state and may trigger a redraw of the screen.
- **Functions called**:
    - [`screen_write_collect_flush`](#screen_write_collect_flush)
    - [`screen_write_initctx`](#screen_write_initctx)


# Function Declarations (Public API)

---
### screen_write_collect_trim<!-- {{#callable_declaration:screen_write_collect_trim}} -->
Trims collected screen write items within a specified range.
- **Description**: This function is used to trim collected screen write items within a specified range on a given line. It is typically called to manage the items in a screen write context, ensuring that only relevant items remain within the specified range. The function must be called with valid indices and a non-null context. It handles items that are entirely before, after, or within the specified range, and adjusts or removes them accordingly. If an item is wrapped and starts at the beginning of the line, the wrapped flag is set if the wrapped pointer is not null.
- **Inputs**:
    - `ctx`: A pointer to a screen_write_ctx structure representing the current screen write context. Must not be null.
    - `y`: An unsigned integer representing the line index in the write list. Must be within the valid range of lines in the context.
    - `x`: An unsigned integer representing the starting x-coordinate of the range to trim. Must be within the valid range of the line.
    - `used`: An unsigned integer representing the number of columns used from the starting x-coordinate. Must be a valid range within the line.
    - `wrapped`: A pointer to an integer that will be set to 1 if an item is wrapped and starts at the beginning of the line. Can be null if the wrapped status is not needed.
- **Output**: Returns a pointer to the screen_write_citem structure that is positioned before the trimmed range, or NULL if no such item exists.
- **See also**: [`screen_write_collect_trim`](#screen_write_collect_trim)  (Implementation)


---
### screen_write_collect_clear<!-- {{#callable_declaration:screen_write_collect_clear}} -->
Clears a range of collected screen write lines.
- **Description**: Use this function to clear a specified range of lines in the screen write context's collection list. This is typically used to reset or prepare a section of the screen for new data. It is important to ensure that the context and the specified range are valid and within bounds before calling this function.
- **Inputs**:
    - `ctx`: A pointer to a screen_write_ctx structure representing the current screen write context. Must not be null.
    - `y`: An unsigned integer specifying the starting line index to clear. Must be within the bounds of the context's write list.
    - `n`: An unsigned integer specifying the number of lines to clear starting from the line index y. Must ensure that y + n does not exceed the bounds of the write list.
- **Output**: None
- **See also**: [`screen_write_collect_clear`](#screen_write_collect_clear)  (Implementation)


---
### screen_write_collect_scroll<!-- {{#callable_declaration:screen_write_collect_scroll}} -->
Scrolls the screen content upwards within a specified region.
- **Description**: This function is used to scroll the content of a screen upwards within a defined region, moving each line up by one position and clearing the bottom line. It is typically called when the screen needs to be updated to reflect new content that pushes existing content out of view. The function should be used when the screen context is properly initialized and the region to be scrolled is correctly set. It modifies the screen's write list to reflect the new positions of the lines and clears the bottom line with the specified background color.
- **Inputs**:
    - `ctx`: A pointer to a screen_write_ctx structure representing the current screen writing context. Must not be null, and the context should be properly initialized before calling this function.
    - `bg`: An unsigned integer representing the background color to be used for clearing the bottom line after scrolling. The value should be a valid color code.
- **Output**: None
- **See also**: [`screen_write_collect_scroll`](#screen_write_collect_scroll)  (Implementation)


---
### screen_write_collect_flush<!-- {{#callable_declaration:screen_write_collect_flush}} -->
Flushes collected screen write operations.
- **Description**: This function is used to flush any collected screen write operations, ensuring that all pending changes are applied to the screen. It should be called when you want to ensure that all buffered operations are executed, particularly after a series of write operations that may have been deferred for performance reasons. The function handles scrolling if necessary and resets the background color to its default. It is important to call this function to maintain the integrity of the screen's visual state.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure representing the current screen write context. This must not be null and should be properly initialized before calling this function.
    - `scroll_only`: An integer flag indicating whether to perform only scrolling operations. If non-zero, only scrolling is processed, and other write operations are skipped.
    - `from`: A constant character pointer used for logging purposes to indicate the source of the flush operation. This should be a valid, null-terminated string.
- **Output**: None
- **See also**: [`screen_write_collect_flush`](#screen_write_collect_flush)  (Implementation)


---
### screen_write_overwrite<!-- {{#callable_declaration:screen_write_overwrite}} -->
Overwrites grid cells in a screen context.
- **Description**: This function is used to overwrite grid cells in a screen context, particularly handling cases where UTF-8 characters or padding cells are involved. It ensures that any padding cells associated with wide characters are properly cleared and overwritten. This function should be called when you need to update the screen with new character data, taking into account the complexities of multi-width characters and padding. It returns a non-zero value if any cells were overwritten, indicating that the screen was modified.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure representing the screen context where the overwrite will occur. Must not be null.
    - `gc`: A pointer to a `grid_cell` structure representing the cell data to be written. Must not be null.
    - `width`: An unsigned integer representing the width of the character to be written. Must be greater than zero.
- **Output**: Returns an integer indicating whether any cells were overwritten (non-zero) or not (zero).
- **See also**: [`screen_write_overwrite`](#screen_write_overwrite)  (Implementation)


---
### screen_write_combine<!-- {{#callable_declaration:screen_write_combine}} -->
Combines a UTF-8 zero-width character with the previous character on the screen.
- **Description**: This function attempts to combine a UTF-8 zero-width character with the preceding character in the screen's grid. It is used when writing characters that modify or are visually combined with the previous character, such as zero-width joiners or variation selectors. The function should be called when a character that requires combination is encountered. It handles cases where the character cannot be combined by returning a specific value, and it ensures that the screen's grid is updated correctly. The function must be called with a valid screen write context and grid cell, and it assumes that the cursor is positioned correctly for the combination attempt.
- **Inputs**:
    - `ctx`: A pointer to a screen_write_ctx structure representing the current screen write context. Must not be null.
    - `gc`: A pointer to a grid_cell structure containing the character data to be combined. Must not be null and should represent a character that can be combined with the previous character.
- **Output**: Returns 1 if the character was successfully combined or discarded due to being zero-width; otherwise, returns 0 if the character could not be combined.
- **See also**: [`screen_write_combine`](#screen_write_combine)  (Implementation)


