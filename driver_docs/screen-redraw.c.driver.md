# Purpose
The provided C source code is part of the `tmux` terminal multiplexer, specifically handling the screen redraw functionality. This code is responsible for managing the visual representation of the terminal panes, borders, status lines, and scrollbars within a `tmux` session. It includes functions to determine the type and position of borders, check if a cell is part of a pane or a border, and update the visual elements of the terminal interface based on the current state of the session and its panes.

Key components of this code include functions for setting up the redraw context ([`screen_redraw_set_context`](#screen_redraw_set_context)), updating the status line ([`screen_redraw_update`](#screen_redraw_update)), and drawing various elements such as borders ([`screen_redraw_draw_borders`](#screen_redraw_draw_borders)), panes ([`screen_redraw_draw_panes`](#screen_redraw_draw_panes)), and status lines ([`screen_redraw_draw_status`](#screen_redraw_draw_status)). The code also handles the drawing of scrollbars for panes, which is particularly useful for navigating through content that exceeds the visible area. The code is structured to be integrated into the larger `tmux` application, interacting with other components like the client, session, and window structures to ensure that the terminal display is accurately and efficiently updated.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Data Structures

---
### screen_redraw_border_type
- **Type**: `enum`
- **Members**:
    - `SCREEN_REDRAW_OUTSIDE`: Represents a border type outside the pane.
    - `SCREEN_REDRAW_INSIDE`: Represents a border type inside the pane.
    - `SCREEN_REDRAW_BORDER_LEFT`: Represents the left border of the pane.
    - `SCREEN_REDRAW_BORDER_RIGHT`: Represents the right border of the pane.
    - `SCREEN_REDRAW_BORDER_TOP`: Represents the top border of the pane.
    - `SCREEN_REDRAW_BORDER_BOTTOM`: Represents the bottom border of the pane.
- **Description**: The `screen_redraw_border_type` is an enumeration that defines different types of borders in relation to a pane within a screen. It is used to identify whether a particular cell is inside the pane, outside the pane, or on one of the specific borders (left, right, top, or bottom) of the pane. This enumeration is crucial for rendering the borders correctly when redrawing the screen in applications like terminal multiplexers.


# Functions

---
### screen_redraw_border_set<!-- {{#callable:screen_redraw_border_set}} -->
The `screen_redraw_border_set` function sets the border character and attributes for a grid cell based on the pane line style and cell type.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window containing the pane.
    - `wp`: A pointer to the `window_pane` structure representing the specific pane within the window.
    - `pane_lines`: An enumeration value of type `pane_lines` that specifies the style of the pane lines (e.g., number, double, heavy, simple).
    - `cell_type`: An integer representing the type of cell, which determines how the border is drawn (e.g., inside, outside).
    - `gc`: A pointer to a `grid_cell` structure where the border character and attributes will be set.
- **Control Flow**:
    - Check if `cell_type` is `CELL_OUTSIDE` and if the window's `fill_character` is not NULL; if so, copy the fill character to `gc->data` and return.
    - Use a switch statement to handle different `pane_lines` styles:
    - For `PANE_LINES_NUMBER`, set or clear the `GRID_ATTR_CHARSET` attribute based on `cell_type`, and set the character in `gc->data` to a number or asterisk based on the pane index.
    - For `PANE_LINES_DOUBLE`, clear the `GRID_ATTR_CHARSET` attribute and copy the double border character to `gc->data`.
    - For `PANE_LINES_HEAVY`, clear the `GRID_ATTR_CHARSET` attribute and copy the heavy border character to `gc->data`.
    - For `PANE_LINES_SIMPLE`, clear the `GRID_ATTR_CHARSET` attribute and set the simple border character in `gc->data`.
    - For the default case, set the `GRID_ATTR_CHARSET` attribute and set the character in `gc->data` to the standard border character.
- **Output**:
    - The function does not return a value; it modifies the `grid_cell` structure pointed to by `gc` to set the appropriate border character and attributes.
- **Functions called**:
    - [`utf8_copy`](utf8.c.driver.md#utf8_copy)
    - [`utf8_set`](utf8.c.driver.md#utf8_set)
    - [`window_pane_index`](window.c.driver.md#window_pane_index)
    - [`tty_acs_double_borders`](tty-acs.c.driver.md#tty_acs_double_borders)
    - [`tty_acs_heavy_borders`](tty-acs.c.driver.md#tty_acs_heavy_borders)


---
### screen_redraw_two_panes<!-- {{#callable:screen_redraw_two_panes}} -->
The function `screen_redraw_two_panes` checks if a window has exactly two panes and if the second pane is positioned in a specific direction.
- **Inputs**:
    - `w`: A pointer to a `struct window` which contains a list of panes.
    - `direction`: An integer indicating the direction to check for the second pane's position; 0 for horizontal (x-axis) and 1 for vertical (y-axis).
- **Control Flow**:
    - Retrieve the first pane from the window's pane list and get the next pane in the list.
    - If there is no second pane, return 0 indicating there is only one pane.
    - If there is a third pane, return 0 indicating there are more than two panes.
    - If `direction` is 0 and the x-offset of the second pane is 0, return 0 indicating the second pane is not positioned horizontally.
    - If `direction` is 1 and the y-offset of the second pane is 0, return 0 indicating the second pane is not positioned vertically.
    - Return 1 if the window has exactly two panes and the second pane is positioned in the specified direction.
- **Output**:
    - Returns an integer: 1 if the window has exactly two panes and the second pane is positioned in the specified direction, otherwise 0.


---
### screen_redraw_pane_border<!-- {{#callable:screen_redraw_pane_border}} -->
The `screen_redraw_pane_border` function determines the type of border a given cell (px, py) is in relation to a specific window pane, considering various factors like pane position, scrollbars, and split indicators.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains context information for the screen redraw operation, including pane status and scrollbar settings.
    - `wp`: A pointer to a `window_pane` structure, representing the pane for which the border type is being determined.
    - `px`: An unsigned integer representing the x-coordinate of the cell being checked.
    - `py`: An unsigned integer representing the y-coordinate of the cell being checked.
- **Control Flow**:
    - Check if the cell (px, py) is inside the pane's boundaries; if so, return `SCREEN_REDRAW_INSIDE`.
    - Retrieve pane border indicators from the pane's options and determine if the window is split horizontally or vertically.
    - Check if scrollbars are enabled for the pane and calculate the scrollbar width if they are.
    - Determine if the cell is on the left or right border of the pane, considering scrollbar position and split indicators.
    - Determine if the cell is on the top or bottom border of the pane, considering split indicators and scrollbar position.
    - If none of the above conditions are met, return `SCREEN_REDRAW_OUTSIDE` indicating the cell is outside the pane.
- **Output**:
    - Returns an enum value of type `screen_redraw_border_type`, indicating the border type of the cell in relation to the pane (e.g., inside, outside, left border, right border, top border, or bottom border).
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`screen_redraw_two_panes`](#screen_redraw_two_panes)
    - [`window_pane_show_scrollbar`](window.c.driver.md#window_pane_show_scrollbar)


---
### screen_redraw_cell_border<!-- {{#callable:screen_redraw_cell_border}} -->
The `screen_redraw_cell_border` function checks if a given cell position (px, py) is on the border of a window or any of its panes within a screen redraw context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains the context for the screen redraw operation, including the client and pane status.
    - `px`: An unsigned integer representing the x-coordinate of the cell to check.
    - `py`: An unsigned integer representing the y-coordinate of the cell to check.
- **Control Flow**:
    - Retrieve the client and current window from the context.
    - Adjust the window's height if the pane status is set to bottom.
    - Check if the cell is outside the window's dimensions; if so, return 0.
    - Check if the cell is on the window's border; if so, return 1.
    - Iterate over each pane in the window using a TAILQ_FOREACH loop.
    - For each visible pane, call [`screen_redraw_pane_border`](#screen_redraw_pane_border) to determine the cell's relation to the pane.
    - If the cell is inside a pane, return 0.
    - If the cell is on a pane's border, return 1.
    - If none of the conditions are met, return 0.
- **Output**:
    - Returns an integer: 1 if the cell is on a border, 0 otherwise.
- **Functions called**:
    - [`window_pane_visible`](window.c.driver.md#window_pane_visible)
    - [`screen_redraw_pane_border`](#screen_redraw_pane_border)


---
### screen_redraw_type_of_cell<!-- {{#callable:screen_redraw_type_of_cell}} -->
The function `screen_redraw_type_of_cell` determines the type of border cell at a given position in a window based on its surrounding cells.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure that contains context information for the screen redraw operation, including the client and pane status.
    - `px`: An unsigned integer representing the x-coordinate of the cell within the window.
    - `py`: An unsigned integer representing the y-coordinate of the cell within the window.
- **Control Flow**:
    - Retrieve the client and window dimensions from the context structure.
    - Adjust the window's height if the pane status is set to `PANE_STATUS_BOTTOM`.
    - Check if the given cell coordinates (px, py) are outside the window dimensions; if so, return `CELL_OUTSIDE`.
    - Initialize a bitmask to determine which of the surrounding cells are borders.
    - Check the left, right, top, and bottom cells relative to the current cell to update the bitmask with border information.
    - Use a switch statement to determine the type of border cell based on the bitmask value, returning the appropriate cell type constant.
    - If no specific border type is matched, return `CELL_OUTSIDE`.
- **Output**:
    - The function returns an integer representing the type of border cell, such as `CELL_JOIN`, `CELL_BOTTOMJOIN`, `CELL_TOPJOIN`, etc., or `CELL_OUTSIDE` if the cell is outside the window.
- **Functions called**:
    - [`screen_redraw_cell_border`](#screen_redraw_cell_border)


---
### screen_redraw_check_cell<!-- {{#callable:screen_redraw_check_cell}} -->
The function `screen_redraw_check_cell` determines the type of cell at a given position in a window, considering pane visibility, borders, and scrollbars.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains context information for screen redraw operations, including client and pane status.
    - `px`: An unsigned integer representing the x-coordinate of the cell to check.
    - `py`: An unsigned integer representing the y-coordinate of the cell to check.
    - `wpp`: A pointer to a pointer to a `window_pane` structure, which will be set to the pane containing the cell if applicable.
- **Control Flow**:
    - Initialize local variables and set `*wpp` to NULL.
    - Check if the cell is outside the window dimensions; if so, return `CELL_OUTSIDE`.
    - Check if the cell is on the window border; if so, return the type of border cell using [`screen_redraw_type_of_cell`](#screen_redraw_type_of_cell).
    - If pane status is not off, iterate over panes to check if the cell is within a pane's status line; if so, return `CELL_INSIDE`.
    - Iterate over panes to check if the cell is within a visible pane or its scrollbar; if so, return `CELL_INSIDE` or `CELL_SCROLLBAR` as appropriate.
    - If the cell is on a pane border, determine the type of border cell and return it.
    - If no conditions are met, return `CELL_OUTSIDE`.
- **Output**:
    - Returns an integer indicating the type of cell: `CELL_OUTSIDE`, `CELL_INSIDE`, `CELL_SCROLLBAR`, or a specific border type.
- **Functions called**:
    - [`screen_redraw_type_of_cell`](#screen_redraw_type_of_cell)
    - [`server_client_get_pane`](server-client.c.driver.md#server_client_get_pane)
    - [`window_pane_visible`](window.c.driver.md#window_pane_visible)
    - [`window_pane_show_scrollbar`](window.c.driver.md#window_pane_show_scrollbar)
    - [`screen_redraw_pane_border`](#screen_redraw_pane_border)


---
### screen_redraw_check_is<!-- {{#callable:screen_redraw_check_is}} -->
The `screen_redraw_check_is` function determines if a specific cell in a window pane is on the border of the pane.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains context information for the screen redraw operation.
    - `px`: An unsigned integer representing the x-coordinate of the cell to check.
    - `py`: An unsigned integer representing the y-coordinate of the cell to check.
    - `wp`: A pointer to a `window_pane` structure, representing the pane in which the cell is located.
- **Control Flow**:
    - The function calls [`screen_redraw_pane_border`](#screen_redraw_pane_border) with the provided context, pane, and cell coordinates to determine the border type of the cell.
    - It checks if the returned border type is neither `SCREEN_REDRAW_INSIDE` nor `SCREEN_REDRAW_OUTSIDE`.
    - If the border type is not inside or outside, it returns 1, indicating the cell is on the border.
    - Otherwise, it returns 0, indicating the cell is not on the border.
- **Output**:
    - The function returns an integer: 1 if the cell is on the border of the pane, and 0 otherwise.
- **Functions called**:
    - [`screen_redraw_pane_border`](#screen_redraw_pane_border)


---
### screen_redraw_make_pane_status<!-- {{#callable:screen_redraw_make_pane_status}} -->
The function `screen_redraw_make_pane_status` updates the status line of a specific window pane and determines if a redraw is necessary based on changes.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client for which the pane status is being updated.
    - `wp`: A pointer to the `window_pane` structure representing the specific pane whose status is being updated.
    - `rctx`: A pointer to the `screen_redraw_ctx` structure containing the context for the screen redraw operation, including pane status and scrollbars information.
    - `pane_lines`: An enumeration value of type `pane_lines` indicating the style of pane lines to be used for the border.
- **Control Flow**:
    - Initialize local variables and determine if scrollbars are shown for the pane, adjusting the scrollbar width accordingly.
    - Create a format tree for the pane and apply the appropriate border style based on whether the pane is active or not.
    - Expand the pane border format string using the format tree and calculate the status line width based on the pane's size and scrollbar width.
    - Initialize a new screen for the pane's status line and start writing to it.
    - Iterate over each cell in the status line width, determining the type of cell and setting the border style accordingly, then write the cell to the screen.
    - Clear the character set attribute from the grid cell and draw the expanded format string onto the status line screen.
    - Stop writing to the screen and free the expanded format string and format tree.
    - Compare the new status screen grid with the old one to determine if there are changes, freeing the old screen and returning 1 if changes are detected, otherwise returning 0.
- **Output**:
    - Returns an integer indicating whether the pane status screen has changed (1) or not (0).
- **Functions called**:
    - [`window_pane_show_scrollbar`](window.c.driver.md#window_pane_show_scrollbar)
    - [`format_create`](format.c.driver.md#format_create)
    - [`format_defaults`](format.c.driver.md#format_defaults)
    - [`server_client_get_pane`](server-client.c.driver.md#server_client_get_pane)
    - [`style_apply`](style.c.driver.md#style_apply)
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`format_expand_time`](format.c.driver.md#format_expand_time)
    - [`screen_init`](screen.c.driver.md#screen_init)
    - [`screen_write_start`](screen-write.c.driver.md#screen_write_start)
    - [`screen_redraw_type_of_cell`](#screen_redraw_type_of_cell)
    - [`screen_redraw_border_set`](#screen_redraw_border_set)
    - [`screen_write_cell`](screen-write.c.driver.md#screen_write_cell)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`format_draw`](format-draw.c.driver.md#format_draw)
    - [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop)
    - [`format_free`](format.c.driver.md#format_free)
    - [`grid_compare`](grid.c.driver.md#grid_compare)
    - [`screen_free`](screen.c.driver.md#screen_free)


---
### screen_redraw_draw_pane_status<!-- {{#callable:screen_redraw_draw_pane_status}} -->
The function `screen_redraw_draw_pane_status` is responsible for drawing the status line for each visible pane in a window, adjusting for visibility and position within the terminal display.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains context information for the screen redraw operation, including the client, window, and pane status details.
- **Control Flow**:
    - Retrieve the client and current window from the context.
    - Iterate over each pane in the window using a TAILQ_FOREACH loop.
    - For each pane, check if it is visible using [`window_pane_visible`](window.c.driver.md#window_pane_visible). If not, continue to the next pane.
    - Determine the status screen and size for the pane.
    - Calculate the y-offset based on whether the pane status is at the top or bottom.
    - Calculate the x-offset by adding a fixed offset to the pane's x-offset.
    - Check if the pane's status line is within the visible area of the terminal using the context's offset and size values.
    - Determine the portion of the status line that is visible and set the starting index, x position, and width accordingly.
    - If the status line is at the top, adjust the y-offset by adding the number of status lines.
    - Draw the visible portion of the status line using [`tty_draw_line`](tty.c.driver.md#tty_draw_line).
    - Reset the cursor position to the top-left corner of the terminal using [`tty_cursor`](tty.c.driver.md#tty_cursor).
- **Output**:
    - The function does not return a value; it performs drawing operations directly on the terminal associated with the client.
- **Functions called**:
    - [`window_pane_visible`](window.c.driver.md#window_pane_visible)
    - [`tty_draw_line`](tty.c.driver.md#tty_draw_line)
    - [`tty_cursor`](tty.c.driver.md#tty_cursor)


---
### screen_redraw_update<!-- {{#callable:screen_redraw_update}} -->
The `screen_redraw_update` function updates the redraw flags for a client's screen based on the current status and pane conditions.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure that contains context information for the screen redraw operation, including the client and pane status.
    - `flags`: A 64-bit unsigned integer representing the current redraw flags that determine which parts of the screen need to be updated.
- **Control Flow**:
    - Retrieve the client and current window from the context structure.
    - Check if there is a message or prompt string for the client and set the redraw flag accordingly using [`status_message_redraw`](status.c.driver.md#status_message_redraw), [`status_prompt_redraw`](status.c.driver.md#status_prompt_redraw), or [`status_redraw`](status.c.driver.md#status_redraw).
    - If no redraw is needed and the `CLIENT_REDRAWSTATUSALWAYS` flag is not set, clear the `CLIENT_REDRAWSTATUS` flag from the input flags.
    - If there is an overlay draw function for the client, set the `CLIENT_REDRAWOVERLAY` flag.
    - If pane status is not off, iterate over each pane in the current window and update the pane status using [`screen_redraw_make_pane_status`](#screen_redraw_make_pane_status).
    - If any pane status was updated, set the `CLIENT_REDRAWBORDERS` flag.
    - Return the updated flags.
- **Output**:
    - The function returns a 64-bit unsigned integer representing the updated redraw flags, indicating which parts of the screen need to be redrawn.
- **Functions called**:
    - [`status_message_redraw`](status.c.driver.md#status_message_redraw)
    - [`status_prompt_redraw`](status.c.driver.md#status_prompt_redraw)
    - [`status_redraw`](status.c.driver.md#status_redraw)
    - [`screen_redraw_make_pane_status`](#screen_redraw_make_pane_status)


---
### screen_redraw_set_context<!-- {{#callable:screen_redraw_set_context}} -->
The function `screen_redraw_set_context` initializes and sets up the screen redraw context for a given client, preparing it for rendering operations.
- **Inputs**:
    - `c`: A pointer to a `struct client`, representing the client for which the screen redraw context is being set.
    - `ctx`: A pointer to a `struct screen_redraw_ctx`, which will be initialized and populated with context information for screen redraw operations.
- **Control Flow**:
    - Initialize the `ctx` structure to zero using `memset` and set the client pointer `ctx->c` to the provided client `c`.
    - Retrieve the number of status lines using [`status_line_size`](status.c.driver.md#status_line_size) and adjust if there is a message or prompt string present.
    - Determine if the status line should be at the top by checking the 'status-position' option and set `ctx->statustop` accordingly.
    - Set `ctx->statuslines` to the calculated number of status lines.
    - Retrieve and set pane-related options such as `pane-border-status`, `pane-border-lines`, `pane-scrollbars`, and `pane-scrollbars-position` from the window's options.
    - Calculate the window offset and size using [`tty_window_offset`](tty.c.driver.md#tty_window_offset) and store these values in `ctx->ox`, `ctx->oy`, `ctx->sx`, and `ctx->sy`.
    - Log the context setup details for debugging purposes.
- **Output**:
    - The function does not return a value; it modifies the `ctx` structure in place to contain the necessary context for screen redraw operations.
- **Functions called**:
    - [`status_line_size`](status.c.driver.md#status_line_size)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`tty_window_offset`](tty.c.driver.md#tty_window_offset)


---
### screen_redraw_screen<!-- {{#callable:screen_redraw_screen}} -->
The `screen_redraw_screen` function is responsible for redrawing the entire screen for a given client, updating various components like borders, panes, status lines, and overlays based on specific flags and conditions.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client for which the screen needs to be redrawn.
- **Control Flow**:
    - Check if the client is suspended; if so, return immediately without redrawing.
    - Initialize a `screen_redraw_ctx` structure by calling [`screen_redraw_set_context`](#screen_redraw_set_context) to set up the redraw context for the client.
    - Call [`screen_redraw_update`](#screen_redraw_update) to determine which parts of the screen need to be redrawn and update the flags accordingly.
    - If no redraw flags are set, return without further action.
    - Begin synchronizing the terminal by calling [`tty_sync_start`](tty.c.driver.md#tty_sync_start) and update the terminal mode with [`tty_update_mode`](tty.c.driver.md#tty_update_mode).
    - If the redraw flags indicate that borders or windows need to be redrawn, call [`screen_redraw_draw_borders`](#screen_redraw_draw_borders), [`screen_redraw_draw_pane_status`](#screen_redraw_draw_pane_status), and [`screen_redraw_draw_pane_scrollbars`](#screen_redraw_draw_pane_scrollbars) as needed.
    - If the redraw flags indicate that panes need to be redrawn, call [`screen_redraw_draw_panes`](#screen_redraw_draw_panes) and [`screen_redraw_draw_pane_scrollbars`](#screen_redraw_draw_pane_scrollbars).
    - If status lines need to be redrawn, call [`screen_redraw_draw_status`](#screen_redraw_draw_status).
    - If an overlay needs to be redrawn, call the client's `overlay_draw` function with the appropriate context.
    - Finally, reset the terminal state with [`tty_reset`](tty.c.driver.md#tty_reset).
- **Output**:
    - The function does not return a value; it performs screen updates directly on the client's terminal.
- **Functions called**:
    - [`screen_redraw_set_context`](#screen_redraw_set_context)
    - [`screen_redraw_update`](#screen_redraw_update)
    - [`tty_sync_start`](tty.c.driver.md#tty_sync_start)
    - [`tty_update_mode`](tty.c.driver.md#tty_update_mode)
    - [`screen_redraw_draw_borders`](#screen_redraw_draw_borders)
    - [`screen_redraw_draw_pane_status`](#screen_redraw_draw_pane_status)
    - [`screen_redraw_draw_pane_scrollbars`](#screen_redraw_draw_pane_scrollbars)
    - [`screen_redraw_draw_panes`](#screen_redraw_draw_panes)
    - [`screen_redraw_draw_status`](#screen_redraw_draw_status)
    - [`tty_reset`](tty.c.driver.md#tty_reset)


---
### screen_redraw_pane<!-- {{#callable:screen_redraw_pane}} -->
The `screen_redraw_pane` function redraws a specific window pane and optionally its scrollbar for a given client.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the pane is being redrawn.
    - `wp`: A pointer to a `window_pane` structure representing the pane that needs to be redrawn.
    - `redraw_scrollbar_only`: An integer flag indicating whether only the scrollbar should be redrawn (non-zero value) or the entire pane should be redrawn (zero value).
- **Control Flow**:
    - Check if the window pane `wp` is visible using `window_pane_visible(wp)`. If not, return immediately.
    - Initialize a `screen_redraw_ctx` structure `ctx` by calling `screen_redraw_set_context(c, &ctx)`.
    - Start synchronizing the terminal state with `tty_sync_start(&c->tty)` and update the terminal mode with `tty_update_mode(&c->tty, c->tty.mode, NULL)`.
    - If `redraw_scrollbar_only` is false, call `screen_redraw_draw_pane(&ctx, wp)` to redraw the entire pane.
    - Check if the pane's scrollbar should be shown using `window_pane_show_scrollbar(wp, ctx.pane_scrollbars)`. If true, call `screen_redraw_draw_pane_scrollbar(&ctx, wp)` to draw the scrollbar.
    - Reset the terminal state with `tty_reset(&c->tty)` to conclude the drawing process.
- **Output**:
    - The function does not return any value; it performs the side effect of redrawing the specified pane and its scrollbar on the client's terminal.
- **Functions called**:
    - [`window_pane_visible`](window.c.driver.md#window_pane_visible)
    - [`screen_redraw_set_context`](#screen_redraw_set_context)
    - [`tty_sync_start`](tty.c.driver.md#tty_sync_start)
    - [`tty_update_mode`](tty.c.driver.md#tty_update_mode)
    - [`screen_redraw_draw_pane`](#screen_redraw_draw_pane)
    - [`window_pane_show_scrollbar`](window.c.driver.md#window_pane_show_scrollbar)
    - [`screen_redraw_draw_pane_scrollbar`](#screen_redraw_draw_pane_scrollbar)
    - [`tty_reset`](tty.c.driver.md#tty_reset)


---
### screen_redraw_draw_borders_style<!-- {{#callable:screen_redraw_draw_borders_style}} -->
The function `screen_redraw_draw_borders_style` determines and applies the appropriate border style for a window pane based on its active status and returns the grid cell style for the border.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains context information for the screen redraw operation, including the client and session details.
    - `x`: An unsigned integer representing the x-coordinate of the border cell being styled.
    - `y`: An unsigned integer representing the y-coordinate of the border cell being styled.
    - `wp`: A pointer to a `window_pane` structure, representing the specific pane for which the border style is being determined.
- **Control Flow**:
    - Check if the border style for the window pane (`wp`) has already been set by examining `wp->border_gc_set`.
    - If the border style is already set, return the existing `border_gc`.
    - If not set, mark the border style as set by setting `wp->border_gc_set` to 1.
    - Create a default format tree (`ft`) using [`format_create_defaults`](format.c.driver.md#format_create_defaults) with the current client, session, window, and pane.
    - Check if the current cell (x, y) is on the border of the active pane using [`screen_redraw_check_is`](#screen_redraw_check_is).
    - If the cell is on the active pane's border, apply the 'pane-active-border-style' to `wp->border_gc` using [`style_apply`](style.c.driver.md#style_apply).
    - If the cell is not on the active pane's border, apply the 'pane-border-style' to `wp->border_gc`.
    - Free the format tree (`ft`) using [`format_free`](format.c.driver.md#format_free).
    - Return the pointer to the `border_gc` of the window pane.
- **Output**:
    - Returns a pointer to a `grid_cell` structure representing the style of the border for the specified window pane.
- **Functions called**:
    - [`server_client_get_pane`](server-client.c.driver.md#server_client_get_pane)
    - [`format_create_defaults`](format.c.driver.md#format_create_defaults)
    - [`screen_redraw_check_is`](#screen_redraw_check_is)
    - [`style_apply`](style.c.driver.md#style_apply)
    - [`format_free`](format.c.driver.md#format_free)


---
### screen_redraw_draw_borders_cell<!-- {{#callable:screen_redraw_draw_borders_cell}} -->
The function `screen_redraw_draw_borders_cell` is responsible for drawing the border cell of a pane in a terminal screen, considering various conditions like overlay checks, cell type, and border styles.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure that contains context information for the screen redraw operation, including client, session, window, and other relevant data.
    - `i`: An unsigned integer representing the x-coordinate offset within the terminal screen where the border cell is to be drawn.
    - `j`: An unsigned integer representing the y-coordinate offset within the terminal screen where the border cell is to be drawn.
- **Control Flow**:
    - Check if an overlay is present and if the current cell is within the overlay range; if not, return early.
    - Determine the type of cell (e.g., inside, scrollbar, border) using [`screen_redraw_check_cell`](#screen_redraw_check_cell); if it's inside or a scrollbar, return early.
    - If the cell is not associated with a specific pane, set the default pane border style if not already set, and copy it to the grid cell `gc`.
    - If the cell is associated with a pane, retrieve the border style using [`screen_redraw_draw_borders_style`](#screen_redraw_draw_borders_style) and apply it to `gc`.
    - Check if the pane is marked and if the cell is part of the marked pane, then toggle the reverse attribute on `gc`.
    - Set the border character for the cell using [`screen_redraw_border_set`](#screen_redraw_border_set).
    - Determine if the cell should be isolated based on its type and terminal capabilities, and adjust the cursor position accordingly.
    - Check the pane border indicators option to determine if arrows should be drawn on the border.
    - If arrows are enabled and the cell is on a border, set the character set attribute and assign the appropriate border marker to `gc`.
    - Draw the cell on the terminal using [`tty_cell`](tty.c.driver.md#tty_cell), and handle isolation markers if necessary.
- **Output**:
    - The function does not return a value; it performs operations to draw a border cell on the terminal screen.
- **Functions called**:
    - [`server_client_get_pane`](server-client.c.driver.md#server_client_get_pane)
    - [`screen_redraw_check_cell`](#screen_redraw_check_cell)
    - [`format_create_defaults`](format.c.driver.md#format_create_defaults)
    - [`style_add`](style.c.driver.md#style_add)
    - [`format_free`](format.c.driver.md#format_free)
    - [`screen_redraw_draw_borders_style`](#screen_redraw_draw_borders_style)
    - [`server_is_marked`](server.c.driver.md#server_is_marked)
    - [`screen_redraw_check_is`](#screen_redraw_check_is)
    - [`screen_redraw_border_set`](#screen_redraw_border_set)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`tty_cursor`](tty.c.driver.md#tty_cursor)
    - [`tty_puts`](tty.c.driver.md#tty_puts)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`screen_redraw_pane_border`](#screen_redraw_pane_border)
    - [`utf8_set`](utf8.c.driver.md#utf8_set)
    - [`tty_cell`](tty.c.driver.md#tty_cell)


---
### screen_redraw_draw_borders<!-- {{#callable:screen_redraw_draw_borders}} -->
The `screen_redraw_draw_borders` function iterates over the screen area of a client to draw borders around window panes.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains context information for the screen redraw operation, including the client, session, window, and other display parameters.
- **Control Flow**:
    - Retrieve the client, session, and current window from the context structure.
    - Log a debug message with the function name, client name, and window ID.
    - Iterate over each window pane in the current window and reset the `border_gc_set` flag to 0 for each pane.
    - Loop over each row (`j`) from 0 to the height of the terminal minus the number of status lines.
    - Within each row, loop over each column (`i`) from 0 to the width of the terminal.
    - For each cell at position `(i, j)`, call [`screen_redraw_draw_borders_cell`](#screen_redraw_draw_borders_cell) to draw the border cell.
- **Output**:
    - The function does not return a value; it performs its operations directly on the terminal display through the context's client and terminal structures.
- **Functions called**:
    - [`screen_redraw_draw_borders_cell`](#screen_redraw_draw_borders_cell)


---
### screen_redraw_draw_panes<!-- {{#callable:screen_redraw_draw_panes}} -->
The function `screen_redraw_draw_panes` iterates over all panes in the current window and redraws each visible pane using the provided context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains the context for the screen redraw operation, including the client and window information.
- **Control Flow**:
    - Retrieve the current client and window from the context.
    - Log a debug message with the function name, client name, and window ID.
    - Iterate over each pane in the current window using the `TAILQ_FOREACH` macro.
    - For each pane, check if it is visible using [`window_pane_visible`](window.c.driver.md#window_pane_visible).
    - If the pane is visible, call [`screen_redraw_draw_pane`](#screen_redraw_draw_pane) with the context and the pane.
- **Output**:
    - This function does not return a value; it performs the side effect of redrawing visible panes on the screen.
- **Functions called**:
    - [`window_pane_visible`](window.c.driver.md#window_pane_visible)
    - [`screen_redraw_draw_pane`](#screen_redraw_draw_pane)


---
### screen_redraw_draw_status<!-- {{#callable:screen_redraw_draw_status}} -->
The function `screen_redraw_draw_status` is responsible for drawing the status line on the terminal screen for a given client context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure which contains the context for the screen redraw operation, including client, window, and status line information.
- **Control Flow**:
    - Retrieve the client and window from the context structure.
    - Log the function call with the client's name and window ID for debugging purposes.
    - Determine the starting y-coordinate for the status line based on whether the status line is at the top or bottom of the screen.
    - Iterate over each line of the status line (up to `ctx->statuslines`) and draw it using [`tty_draw_line`](tty.c.driver.md#tty_draw_line), specifying the terminal, screen, and line details.
- **Output**:
    - The function does not return a value; it performs the side effect of drawing the status line on the terminal.
- **Functions called**:
    - [`tty_draw_line`](tty.c.driver.md#tty_draw_line)


---
### screen_redraw_draw_pane<!-- {{#callable:screen_redraw_draw_pane}} -->
The `screen_redraw_draw_pane` function is responsible for rendering a specific window pane on the terminal screen, taking into account its visibility and position within the overall screen context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure that contains the context for the screen redraw operation, including the client, offsets, and dimensions of the visible area.
    - `wp`: A pointer to a `window_pane` structure representing the pane to be drawn, including its position, size, and associated screen and palette.
- **Control Flow**:
    - The function begins by logging the function call with the client name and pane ID for debugging purposes.
    - It checks if the pane is outside the visible area of the screen context by comparing the pane's offsets and size with the context's offsets and size; if so, it returns early without drawing.
    - If the status line is at the top, it adjusts the top offset accordingly; otherwise, it sets the top offset to zero.
    - The function iterates over each line (row) of the pane's screen, checking if the line is within the visible vertical range of the context; if not, it skips to the next line.
    - For each visible line, it calculates the starting x-coordinate and width of the line to be drawn, based on the pane's horizontal position relative to the context's visible area.
    - It logs the details of the line being drawn for debugging purposes.
    - The function sets the default colors for the terminal and calls [`tty_draw_line`](tty.c.driver.md#tty_draw_line) to render the line on the terminal, using the pane's screen, calculated coordinates, and default colors.
    - If the `ENABLE_SIXEL` macro is defined, it calls [`tty_draw_images`](tty.c.driver.md#tty_draw_images) to render any images associated with the pane.
- **Output**:
    - The function does not return a value; it performs its operations directly on the terminal output through the `tty` interface.
- **Functions called**:
    - [`tty_default_colours`](tty.c.driver.md#tty_default_colours)
    - [`tty_draw_line`](tty.c.driver.md#tty_draw_line)
    - [`tty_draw_images`](tty.c.driver.md#tty_draw_images)


---
### screen_redraw_draw_pane_scrollbars<!-- {{#callable:screen_redraw_draw_pane_scrollbars}} -->
The function `screen_redraw_draw_pane_scrollbars` iterates over all window panes in the current window and draws scrollbars for those panes that are visible and have scrollbars enabled.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains context information for the screen redraw operation, including the client, window, and pane settings.
- **Control Flow**:
    - Retrieve the current client and window from the context structure.
    - Log a debug message with the function name, client name, and window ID.
    - Iterate over each window pane in the current window using a TAILQ_FOREACH loop.
    - For each pane, check if the pane should show a scrollbar and if it is visible using [`window_pane_show_scrollbar`](window.c.driver.md#window_pane_show_scrollbar) and [`window_pane_visible`](window.c.driver.md#window_pane_visible).
    - If both conditions are true, call [`screen_redraw_draw_pane_scrollbar`](#screen_redraw_draw_pane_scrollbar) to draw the scrollbar for the pane.
- **Output**:
    - This function does not return any value; it performs drawing operations on the screen.
- **Functions called**:
    - [`window_pane_show_scrollbar`](window.c.driver.md#window_pane_show_scrollbar)
    - [`window_pane_visible`](window.c.driver.md#window_pane_visible)
    - [`screen_redraw_draw_pane_scrollbar`](#screen_redraw_draw_pane_scrollbar)


---
### screen_redraw_draw_pane_scrollbar<!-- {{#callable:screen_redraw_draw_pane_scrollbar}} -->
The function `screen_redraw_draw_pane_scrollbar` calculates and draws the scrollbar for a given window pane based on its current mode and position.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure that contains context information for the screen redraw operation, including offsets and scrollbar positions.
    - `wp`: A pointer to a `window_pane` structure representing the pane for which the scrollbar is being drawn.
- **Control Flow**:
    - Retrieve the screen associated with the window pane `wp` and initialize variables for scrollbar dimensions and positions.
    - Check if the window pane is in a mode that does not require a scrollbar; if so, return early if the scrollbar is modal.
    - Calculate the total height of the content and the percentage of the viewable area to determine the height and position of the scrollbar slider.
    - Determine the x-position of the scrollbar based on whether it is positioned on the left or right of the pane.
    - Ensure the slider height is at least 1 and adjust the slider position if it exceeds the scrollbar height.
    - Call [`screen_redraw_draw_scrollbar`](#screen_redraw_draw_scrollbar) to render the scrollbar on the screen with the calculated dimensions and positions.
    - Store the current position and height of the slider in the `window_pane` structure for future reference.
- **Output**:
    - The function does not return a value but updates the display to show the scrollbar for the specified pane and stores the slider's position and height in the `window_pane` structure.
- **Functions called**:
    - [`window_pane_mode`](window.c.driver.md#window_pane_mode)
    - [`window_copy_get_current_offset`](window-copy.c.driver.md#window_copy_get_current_offset)
    - [`screen_redraw_draw_scrollbar`](#screen_redraw_draw_scrollbar)


---
### screen_redraw_draw_scrollbar<!-- {{#callable:screen_redraw_draw_scrollbar}} -->
The function `screen_redraw_draw_scrollbar` draws a scrollbar on a terminal screen for a given window pane, including the slider and padding, based on the specified position and dimensions.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains context information for the screen redraw operation, including the client and terminal dimensions.
    - `wp`: A pointer to a `window_pane` structure, representing the pane for which the scrollbar is being drawn, including its scrollbar style.
    - `sb_pos`: An integer indicating the position of the scrollbar, either left or right of the pane.
    - `sb_x`: An integer representing the x-coordinate where the scrollbar should be drawn.
    - `sb_y`: An integer representing the y-coordinate where the scrollbar should be drawn.
    - `sb_h`: An unsigned integer representing the height of the scrollbar.
    - `slider_h`: An unsigned integer representing the height of the slider within the scrollbar.
    - `slider_y`: An unsigned integer representing the y-coordinate of the top of the slider within the scrollbar.
- **Control Flow**:
    - Initialize local variables and retrieve the client and terminal information from the context.
    - Set up the style for the slider by swapping the foreground and background colors of the scrollbar style.
    - Calculate the maximum width (`imax`) and height (`jmax`) of the scrollbar based on the terminal dimensions and scrollbar position.
    - Iterate over each row (`j`) and column (`i`) within the scrollbar dimensions to determine the position (`px`, `py`) on the terminal.
    - Check if the current position is within the visible area of the terminal and skip drawing if it is outside.
    - Move the terminal cursor to the current position (`px`, `py`).
    - Determine whether to draw the slider or the scrollbar padding based on the current position relative to the slider's position and dimensions.
    - Draw the appropriate cell (either slider or padding) using the terminal's [`tty_cell`](tty.c.driver.md#tty_cell) function.
- **Output**:
    - The function does not return a value; it directly manipulates the terminal display to draw the scrollbar and slider.
- **Functions called**:
    - [`tty_cursor`](tty.c.driver.md#tty_cursor)
    - [`tty_cell`](tty.c.driver.md#tty_cell)


