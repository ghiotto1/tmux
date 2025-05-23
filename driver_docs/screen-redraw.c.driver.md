# Purpose
The provided C source code is part of the `tmux` terminal multiplexer, specifically handling the screen redraw functionality. This code is responsible for managing the visual representation of the terminal panes, borders, status lines, and scrollbars within a `tmux` session. It includes functions to determine the type and position of borders around panes, draw the panes and their contents, and update the status line and scrollbars based on the current state of the terminal and user interactions.

The code defines several static functions that encapsulate specific tasks related to screen rendering, such as [`screen_redraw_draw_borders`](#screen_redraw_draw_borders), [`screen_redraw_draw_panes`](#screen_redraw_draw_panes), and [`screen_redraw_draw_status`](#screen_redraw_draw_status). These functions are used to update the visual elements of the terminal, ensuring that changes in the terminal's state are accurately reflected on the screen. The code also includes logic to handle different types of pane borders, such as single, double, and heavy lines, and to manage the display of scrollbars when necessary. The use of enums and macros helps in managing the different states and types of borders and cells, making the code modular and easier to maintain. Overall, this file is a crucial component of the `tmux` application, providing the necessary functionality to render and update the terminal interface dynamically.
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
    - `SCREEN_REDRAW_BORDER_LEFT`: Represents the left border of a pane.
    - `SCREEN_REDRAW_BORDER_RIGHT`: Represents the right border of a pane.
    - `SCREEN_REDRAW_BORDER_TOP`: Represents the top border of a pane.
    - `SCREEN_REDRAW_BORDER_BOTTOM`: Represents the bottom border of a pane.
- **Description**: The `screen_redraw_border_type` is an enumeration that defines various types of borders in relation to a pane within a screen. It is used to identify whether a particular cell is part of the pane's border and, if so, which part of the border it belongs to (left, right, top, bottom, inside, or outside). This enumeration is crucial for rendering the borders of panes correctly in a terminal multiplexer environment, such as tmux, where panes are divided by borders.


# Functions

---
### screen_redraw_border_set<!-- {{#callable:screen_redraw_border_set}} -->
The `screen_redraw_border_set` function sets the border character and attributes for a grid cell based on the pane line style and cell type.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window containing the pane.
    - `wp`: A pointer to the `window_pane` structure representing the specific pane within the window.
    - `pane_lines`: An enumeration value of type `pane_lines` that specifies the style of the pane lines (e.g., number, double, heavy, simple).
    - `cell_type`: An integer representing the type of cell, which determines the border character to use.
    - `gc`: A pointer to a `grid_cell` structure where the border character and attributes will be set.
- **Control Flow**:
    - Check if the `cell_type` is `CELL_OUTSIDE` and if the window's `fill_character` is not NULL, then copy the fill character to the grid cell and return.
    - Use a switch statement to handle different `pane_lines` styles:
    - For `PANE_LINES_NUMBER`, set the grid cell's attributes and character based on whether the cell is outside or inside, and if inside, use the pane index or a default character.
    - For `PANE_LINES_DOUBLE`, `PANE_LINES_HEAVY`, and `PANE_LINES_SIMPLE`, set the grid cell's attributes and character using predefined border styles.
    - For the default case, set the grid cell's attributes and character using a default border style.
- **Output**:
    - The function modifies the `grid_cell` structure pointed to by `gc` to set the appropriate border character and attributes.


---
### screen_redraw_two_panes<!-- {{#callable:screen_redraw_two_panes}} -->
The function `screen_redraw_two_panes` checks if a window has exactly two panes and if the second pane is positioned according to the specified direction.
- **Inputs**:
    - `w`: A pointer to a `struct window` which contains the panes to be checked.
    - `direction`: An integer indicating the direction to check for the second pane's position; 0 for horizontal (x-axis) and 1 for vertical (y-axis).
- **Control Flow**:
    - Retrieve the second pane in the window using `TAILQ_NEXT` on the first pane.
    - If the second pane is `NULL`, return 0 indicating there is only one pane.
    - Check if there is a third pane by calling `TAILQ_NEXT` on the second pane; if it exists, return 0 indicating there are more than two panes.
    - If `direction` is 0 (horizontal) and the second pane's `xoff` is 0, return 0 indicating the second pane is not positioned horizontally.
    - If `direction` is 1 (vertical) and the second pane's `yoff` is 0, return 0 indicating the second pane is not positioned vertically.
    - Return 1 if the window has exactly two panes and the second pane is positioned according to the specified direction.
- **Output**:
    - Returns an integer: 1 if the window has exactly two panes and the second pane is positioned according to the specified direction, otherwise 0.


---
### screen_redraw_pane_border<!-- {{#callable:screen_redraw_pane_border}} -->
The `screen_redraw_pane_border` function determines the type of border a given cell (px, py) is in relation to a specific window pane, considering factors like pane splits, scrollbars, and pane status.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure that contains context information for the screen redraw operation, including pane status and scrollbar settings.
    - `wp`: A pointer to a `window_pane` structure representing the pane for which the border type is being determined.
    - `px`: An unsigned integer representing the x-coordinate of the cell being checked.
    - `py`: An unsigned integer representing the y-coordinate of the cell being checked.
- **Control Flow**:
    - Check if the cell (px, py) is inside the pane's boundaries; if so, return `SCREEN_REDRAW_INSIDE`.
    - Retrieve pane border indicators from the pane's options and determine if the pane is split horizontally or vertically.
    - Check if scrollbars are enabled for the pane and calculate the scrollbar width if they are.
    - Determine if the cell is on the left or right border of the pane, considering scrollbar position and pane splits.
    - Determine if the cell is on the top or bottom border of the pane, considering vertical splits, pane status, and scrollbar position.
    - If none of the above conditions are met, return `SCREEN_REDRAW_OUTSIDE` indicating the cell is outside the pane.
- **Output**:
    - The function returns an enum value of type `screen_redraw_border_type`, indicating whether the cell is inside, outside, or on a specific border (left, right, top, bottom) of the pane.
- **Functions called**:
    - [`screen_redraw_two_panes`](#screen_redraw_two_panes)


---
### screen_redraw_cell_border<!-- {{#callable:screen_redraw_cell_border}} -->
The `screen_redraw_cell_border` function determines if a given cell in a window is part of the border, either of the window itself or of any pane within the window.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains context information for the screen redraw operation, including the client and pane status.
    - `px`: An unsigned integer representing the x-coordinate of the cell to check.
    - `py`: An unsigned integer representing the y-coordinate of the cell to check.
- **Control Flow**:
    - Retrieve the client and current window from the context structure.
    - Adjust the window's height (`sy`) if the pane status is set to bottom.
    - Check if the cell coordinates (px, py) are outside the window's dimensions; if so, return 0 indicating it's not a border.
    - Check if the cell is on the window's border (right or bottom edge); if so, return 1 indicating it is a border.
    - Iterate over each pane in the window using a TAILQ_FOREACH loop.
    - For each pane, check if it is visible; if not, continue to the next pane.
    - Call [`screen_redraw_pane_border`](#screen_redraw_pane_border) to determine the border status of the cell relative to the current pane.
    - If the cell is inside the pane, return 0 indicating it's not a border.
    - If the cell is outside the pane, continue to the next pane.
    - If the cell is on the border of the pane, return 1 indicating it is a border.
    - If no conditions are met, return 0 indicating the cell is not a border.
- **Output**:
    - Returns an integer: 1 if the cell is on a border, 0 otherwise.
- **Functions called**:
    - [`screen_redraw_pane_border`](#screen_redraw_pane_border)


---
### screen_redraw_type_of_cell<!-- {{#callable:screen_redraw_type_of_cell}} -->
The function `screen_redraw_type_of_cell` determines the type of border cell at a given position in a window based on its surrounding cells.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains context information for the screen redraw operation, including the client and pane status.
    - `px`: An unsigned integer representing the x-coordinate of the cell within the window.
    - `py`: An unsigned integer representing the y-coordinate of the cell within the window.
- **Control Flow**:
    - Retrieve the client and window dimensions from the context structure.
    - Adjust the window's height if the pane status is set to `PANE_STATUS_BOTTOM`.
    - Check if the given cell coordinates (px, py) are outside the window dimensions; if so, return `CELL_OUTSIDE`.
    - Initialize a bitmask to determine which surrounding cells are borders, checking left, right, top, and bottom cells using the [`screen_redraw_cell_border`](#screen_redraw_cell_border) function.
    - Depending on the pane status (top, bottom, or neither), adjust the checks for top and bottom borders accordingly.
    - Use a switch statement to determine the type of border cell based on the bitmask, returning the appropriate cell type constant (e.g., `CELL_JOIN`, `CELL_BOTTOMJOIN`, etc.).
    - If no specific border type is matched, return `CELL_OUTSIDE`.
- **Output**:
    - The function returns an integer representing the type of border cell, such as `CELL_JOIN`, `CELL_BOTTOMJOIN`, `CELL_TOPJOIN`, `CELL_LEFTRIGHT`, `CELL_RIGHTJOIN`, `CELL_BOTTOMRIGHT`, `CELL_TOPRIGHT`, `CELL_LEFTJOIN`, `CELL_BOTTOMLEFT`, `CELL_TOPLEFT`, `CELL_TOPBOTTOM`, or `CELL_OUTSIDE`.
- **Functions called**:
    - [`screen_redraw_cell_border`](#screen_redraw_cell_border)


---
### screen_redraw_check_cell<!-- {{#callable:screen_redraw_check_cell}} -->
The `screen_redraw_check_cell` function determines the type of cell at a given position in a window, considering pane visibility, borders, and scrollbars.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure containing the context for screen redraw operations, including client and pane status information.
    - `px`: An unsigned integer representing the x-coordinate of the cell to check.
    - `py`: An unsigned integer representing the y-coordinate of the cell to check.
    - `wpp`: A pointer to a pointer to a `window_pane` structure, which will be set to the pane containing the cell if applicable.
- **Control Flow**:
    - Initialize local variables and set `*wpp` to NULL.
    - Check if the cell is outside the window dimensions; if so, return `CELL_OUTSIDE`.
    - Check if the cell is on the window border; if so, return the type of border cell using [`screen_redraw_type_of_cell`](#screen_redraw_type_of_cell).
    - If pane status is not off, iterate over panes to check if the cell is within a pane's status line; if so, return `CELL_INSIDE`.
    - Iterate over panes to check if the cell is within a scrollbar; if so, return `CELL_SCROLLBAR`.
    - For each pane, determine if the cell is inside, outside, or on the border using [`screen_redraw_pane_border`](#screen_redraw_pane_border); return the appropriate cell type.
    - If no conditions are met, return `CELL_OUTSIDE`.
- **Output**:
    - Returns an integer indicating the type of cell: `CELL_OUTSIDE`, `CELL_INSIDE`, `CELL_SCROLLBAR`, or a specific border type.
- **Functions called**:
    - [`screen_redraw_type_of_cell`](#screen_redraw_type_of_cell)
    - [`screen_redraw_pane_border`](#screen_redraw_pane_border)


---
### screen_redraw_check_is<!-- {{#callable:screen_redraw_check_is}} -->
The `screen_redraw_check_is` function determines if a specific cell in a window pane is on the border of the pane.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains context information for screen redraw operations.
    - `px`: An unsigned integer representing the x-coordinate of the cell to check.
    - `py`: An unsigned integer representing the y-coordinate of the cell to check.
    - `wp`: A pointer to a `window_pane` structure, representing the pane in which the cell is located.
- **Control Flow**:
    - Call the [`screen_redraw_pane_border`](#screen_redraw_pane_border) function with the provided context, pane, and cell coordinates to determine the border type of the cell.
    - Check if the returned border type is neither `SCREEN_REDRAW_INSIDE` nor `SCREEN_REDRAW_OUTSIDE`.
    - If the border type is not inside or outside, return 1, indicating the cell is on the border.
    - Otherwise, return 0, indicating the cell is not on the border.
- **Output**:
    - Returns an integer: 1 if the cell is on the border of the pane, 0 otherwise.
- **Functions called**:
    - [`screen_redraw_pane_border`](#screen_redraw_pane_border)


---
### screen_redraw_make_pane_status<!-- {{#callable:screen_redraw_make_pane_status}} -->
The function `screen_redraw_make_pane_status` updates the status line of a window pane and determines if a redraw is necessary based on changes.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client for which the pane status is being updated.
    - `wp`: A pointer to the `window_pane` structure representing the specific pane whose status is being updated.
    - `rctx`: A pointer to the `screen_redraw_ctx` structure containing the context for the screen redraw operation.
    - `pane_lines`: An enumeration value of type `pane_lines` indicating the style of pane lines to be used.
- **Control Flow**:
    - Initialize local variables and determine if scrollbars are to be shown for the pane, adjusting the scrollbar width accordingly.
    - Create a format tree for the pane and apply the appropriate border style based on whether the pane is active or not.
    - Expand the pane border format string using the format tree and calculate the width of the status line based on the pane's size and scrollbar width.
    - Initialize a new screen for the pane's status line and start writing to it using a screen write context.
    - Iterate over each cell in the status line width, determining the type of cell and setting the border style accordingly, then write the cell to the screen.
    - Move the cursor to the start of the status line and draw the expanded format string onto the screen.
    - Stop writing to the screen and free the expanded format string and format tree.
    - Compare the new status screen grid with the old one to determine if there are changes, freeing the old screen and returning 1 if changes are detected, otherwise returning 0.
- **Output**:
    - Returns an integer indicating whether the pane status screen has changed (1) or not (0).
- **Functions called**:
    - [`screen_redraw_type_of_cell`](#screen_redraw_type_of_cell)
    - [`screen_redraw_border_set`](#screen_redraw_border_set)


---
### screen_redraw_draw_pane_status<!-- {{#callable:screen_redraw_draw_pane_status}} -->
The function `screen_redraw_draw_pane_status` is responsible for drawing the status line of each visible pane in a window on the terminal screen.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains context information for the screen redraw operation, including the client, window, and terminal dimensions.
- **Control Flow**:
    - Retrieve the current client and window from the context.
    - Iterate over each pane in the window using a TAILQ_FOREACH loop.
    - For each pane, check if it is visible using `window_pane_visible`; if not, continue to the next pane.
    - Determine the status screen and size for the pane.
    - Calculate the y-offset based on whether the pane status is at the top or bottom.
    - Calculate the x-offset for the pane status line.
    - Check if the pane status line is within the visible area of the terminal; if not, continue to the next pane.
    - Determine the portion of the pane status line that is visible and calculate the starting index, x position, and width for drawing.
    - Adjust the y-offset if the status line is at the top of the screen.
    - Draw the visible portion of the pane status line using `tty_draw_line`.
    - Reset the cursor position to (0, 0) using `tty_cursor`.
- **Output**:
    - The function does not return a value; it performs drawing operations directly on the terminal associated with the client.


---
### screen_redraw_update<!-- {{#callable:screen_redraw_update}} -->
The `screen_redraw_update` function updates the redraw flags for a client's screen based on the current status and pane conditions.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure that contains the context for the screen redraw operation, including the client and pane status information.
    - `flags`: A 64-bit unsigned integer representing the current redraw flags that determine which parts of the screen need to be updated.
- **Control Flow**:
    - Retrieve the client and current window from the context.
    - Check if there is a message or prompt string for the client and determine if a status redraw is needed.
    - If no redraw is needed and the `CLIENT_REDRAWSTATUSALWAYS` flag is not set, clear the `CLIENT_REDRAWSTATUS` flag.
    - If there is an overlay draw function, set the `CLIENT_REDRAWOVERLAY` flag.
    - If pane status is not off, iterate over each pane in the window to update pane status and determine if borders need to be redrawn.
    - If any pane status was updated, set the `CLIENT_REDRAWBORDERS` flag.
    - Return the updated flags.
- **Output**:
    - The function returns a 64-bit unsigned integer representing the updated redraw flags, indicating which parts of the screen need to be redrawn.
- **Functions called**:
    - [`screen_redraw_make_pane_status`](#screen_redraw_make_pane_status)


---
### screen_redraw_set_context<!-- {{#callable:screen_redraw_set_context}} -->
The `screen_redraw_set_context` function initializes a `screen_redraw_ctx` structure with the current client's screen redraw settings, including status lines, pane status, and scrollbars.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the current client whose screen redraw context is being set.
    - `ctx`: A pointer to a `screen_redraw_ctx` structure that will be initialized with the current screen redraw settings for the client.
- **Control Flow**:
    - Initialize the `screen_redraw_ctx` structure to zero using `memset` and set the client pointer `ctx->c` to the provided client `c`.
    - Determine the number of status lines using `status_line_size` and adjust if there is a message or prompt string.
    - Set `ctx->statustop` based on the status position option from the session's options.
    - Retrieve pane status, pane lines, pane scrollbars, and scrollbar position from the window's options and store them in the context structure.
    - Calculate the terminal window offset and size using `tty_window_offset` and store these values in the context structure.
    - Log the context settings for debugging purposes.
- **Output**:
    - The function does not return a value; it modifies the `screen_redraw_ctx` structure pointed to by `ctx` to reflect the current screen redraw settings for the client.


---
### screen_redraw_screen<!-- {{#callable:screen_redraw_screen}} -->
The `screen_redraw_screen` function is responsible for redrawing the entire screen for a given client, updating various components like borders, panes, status lines, and overlays based on specific flags and conditions.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the screen needs to be redrawn.
- **Control Flow**:
    - Check if the client is suspended; if so, return immediately without redrawing.
    - Initialize a `screen_redraw_ctx` context structure for the client using [`screen_redraw_set_context`](#screen_redraw_set_context).
    - Update the redraw flags by calling [`screen_redraw_update`](#screen_redraw_update) with the context and client flags.
    - If no redraw flags are set, return without further action.
    - Begin synchronizing the terminal state with `tty_sync_start` and update the terminal mode with `tty_update_mode`.
    - If the redraw flags indicate that borders or windows need redrawing, log the action, draw borders, pane statuses, and scrollbars as necessary.
    - If the redraw flags indicate that panes need redrawing, log the action and draw the panes and their scrollbars.
    - If status lines need redrawing, log the action and draw the status lines.
    - If an overlay needs redrawing, log the action and call the client's overlay draw function.
    - Reset the terminal state with `tty_reset`.
- **Output**:
    - The function does not return a value; it performs screen redrawing operations directly on the client's terminal.
- **Functions called**:
    - [`screen_redraw_set_context`](#screen_redraw_set_context)
    - [`screen_redraw_update`](#screen_redraw_update)
    - [`screen_redraw_draw_borders`](#screen_redraw_draw_borders)
    - [`screen_redraw_draw_pane_status`](#screen_redraw_draw_pane_status)
    - [`screen_redraw_draw_pane_scrollbars`](#screen_redraw_draw_pane_scrollbars)
    - [`screen_redraw_draw_panes`](#screen_redraw_draw_panes)
    - [`screen_redraw_draw_status`](#screen_redraw_draw_status)


---
### screen_redraw_pane<!-- {{#callable:screen_redraw_pane}} -->
The `screen_redraw_pane` function redraws a specific window pane and optionally its scrollbar in a terminal client.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the terminal client where the pane is being redrawn.
    - `wp`: A pointer to a `window_pane` structure representing the specific pane to be redrawn.
    - `redraw_scrollbar_only`: An integer flag indicating whether only the scrollbar should be redrawn (non-zero value) or the entire pane should be redrawn (zero value).
- **Control Flow**:
    - Check if the window pane `wp` is visible using `window_pane_visible`; if not, return immediately.
    - Initialize a `screen_redraw_ctx` structure `ctx` by calling [`screen_redraw_set_context`](#screen_redraw_set_context) with the client `c`.
    - Start synchronizing the terminal state with `tty_sync_start` and update the terminal mode with `tty_update_mode`.
    - If `redraw_scrollbar_only` is false, call [`screen_redraw_draw_pane`](#screen_redraw_draw_pane) to redraw the entire pane.
    - Check if the pane's scrollbar should be shown using `window_pane_show_scrollbar`; if true, call [`screen_redraw_draw_pane_scrollbar`](#screen_redraw_draw_pane_scrollbar) to redraw the scrollbar.
    - Reset the terminal state with `tty_reset`.
- **Output**:
    - The function does not return a value; it performs its operations directly on the terminal client and pane structures.
- **Functions called**:
    - [`screen_redraw_set_context`](#screen_redraw_set_context)
    - [`screen_redraw_draw_pane`](#screen_redraw_draw_pane)
    - [`screen_redraw_draw_pane_scrollbar`](#screen_redraw_draw_pane_scrollbar)


---
### screen_redraw_draw_borders_style<!-- {{#callable:screen_redraw_draw_borders_style}} -->
The function `screen_redraw_draw_borders_style` determines and applies the appropriate border style for a window pane based on its active status and returns the grid cell style for the border.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains context information for the screen redraw operation, including the client and session details.
    - `x`: An unsigned integer representing the x-coordinate of the border cell being styled.
    - `y`: An unsigned integer representing the y-coordinate of the border cell being styled.
    - `wp`: A pointer to a `window_pane` structure, representing the specific pane for which the border style is being determined.
- **Control Flow**:
    - Retrieve the client and session from the context `ctx` and the current window from the session.
    - Check if the border style for the window pane `wp` has already been set (`border_gc_set`).
    - If the border style is already set, return the existing border grid cell style (`border_gc`).
    - Mark the border style as set by setting `border_gc_set` to 1.
    - Create a format tree `ft` with default values using the client, session, current window, and window pane.
    - Check if the current coordinates (x, y) are on the border of the active pane using [`screen_redraw_check_is`](#screen_redraw_check_is).
    - If the coordinates are on the active pane's border, apply the 'pane-active-border-style' to the border grid cell using `style_apply`.
    - If not, apply the 'pane-border-style' to the border grid cell using `style_apply`.
    - Free the format tree `ft` after applying the style.
    - Return the pointer to the border grid cell (`border_gc`).
- **Output**:
    - A pointer to a `grid_cell` structure representing the styled border cell for the window pane.
- **Functions called**:
    - [`screen_redraw_check_is`](#screen_redraw_check_is)


---
### screen_redraw_draw_borders_cell<!-- {{#callable:screen_redraw_draw_borders_cell}} -->
The function `screen_redraw_draw_borders_cell` is responsible for drawing the border cell of a pane in a terminal screen, considering various styles and conditions.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure that contains context information for the screen redraw operation.
    - `i`: An unsigned integer representing the x-coordinate offset for the cell to be drawn.
    - `j`: An unsigned integer representing the y-coordinate offset for the cell to be drawn.
- **Control Flow**:
    - Check if there is an overlay check function in the client context and call it to determine if the cell should be drawn; return if not.
    - Determine the type of cell (inside, scrollbar, or border) using [`screen_redraw_check_cell`](#screen_redraw_check_cell); return if the cell is inside or a scrollbar.
    - If the cell is a border, determine the style to use for the border cell, either using a default style or a specific pane style.
    - If the cell is part of a marked pane, apply a reverse attribute to the grid cell style.
    - Set the border style for the cell using [`screen_redraw_border_set`](#screen_redraw_border_set).
    - Determine if the cell should be isolated based on terminal capabilities and client flags, and adjust the cursor position accordingly.
    - Check if border indicators (arrows) should be drawn and modify the grid cell attributes if necessary.
    - Draw the cell on the terminal using `tty_cell`, and handle isolation markers if needed.
- **Output**:
    - The function does not return a value; it performs operations to draw a border cell on the terminal screen.
- **Functions called**:
    - [`screen_redraw_check_cell`](#screen_redraw_check_cell)
    - [`screen_redraw_draw_borders_style`](#screen_redraw_draw_borders_style)
    - [`screen_redraw_check_is`](#screen_redraw_check_is)
    - [`screen_redraw_border_set`](#screen_redraw_border_set)
    - [`screen_redraw_pane_border`](#screen_redraw_pane_border)


---
### screen_redraw_draw_borders<!-- {{#callable:screen_redraw_draw_borders}} -->
The `screen_redraw_draw_borders` function is responsible for drawing the borders around the panes of a window in a terminal client session.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains context information for the screen redraw operation, including the client, session, window, and other relevant parameters.
- **Control Flow**:
    - Retrieve the client, session, and current window from the context structure.
    - Log a debug message with the function name, client name, and window ID.
    - Iterate over each pane in the window and set the `border_gc_set` flag to 0 for each pane, indicating that the border graphics context is not set.
    - Loop over each row (`j`) from 0 to the height of the terminal minus the number of status lines.
    - Within each row, loop over each column (`i`) from 0 to the width of the terminal.
    - For each cell at position `(i, j)`, call [`screen_redraw_draw_borders_cell`](#screen_redraw_draw_borders_cell) to draw the border cell.
- **Output**:
    - The function does not return a value; it performs its operations directly on the terminal display through the context provided.
- **Functions called**:
    - [`screen_redraw_draw_borders_cell`](#screen_redraw_draw_borders_cell)


---
### screen_redraw_draw_panes<!-- {{#callable:screen_redraw_draw_panes}} -->
The `screen_redraw_draw_panes` function iterates over all panes in the current window and redraws each visible pane using the provided context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains the context for the screen redraw operation, including the client and window information.
- **Control Flow**:
    - Retrieve the current client and window from the context.
    - Log a debug message with the function name, client name, and window ID.
    - Iterate over each pane in the current window using a `TAILQ_FOREACH` loop.
    - For each pane, check if it is visible using the `window_pane_visible` function.
    - If the pane is visible, call [`screen_redraw_draw_pane`](#screen_redraw_draw_pane) with the context and the pane to redraw it.
- **Output**:
    - This function does not return a value; it performs the side effect of redrawing visible panes on the screen.
- **Functions called**:
    - [`screen_redraw_draw_pane`](#screen_redraw_draw_pane)


---
### screen_redraw_draw_status<!-- {{#callable:screen_redraw_draw_status}} -->
The function `screen_redraw_draw_status` is responsible for drawing the status line on the terminal screen for a given client context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains the context for the screen redraw operation, including the client, window, and status line information.
- **Control Flow**:
    - Retrieve the client `c` and the current window `w` from the context `ctx`.
    - Determine the terminal `tty` and the active screen `s` for the status line from the client `c`.
    - Log a debug message with the function name, client name, and window ID.
    - Set the initial y-coordinate `y` for drawing the status line based on whether the status line is at the top or bottom of the screen.
    - Iterate over each line of the status line (`ctx->statuslines`) and draw it using `tty_draw_line`, starting from the calculated y-coordinate `y`.
- **Output**:
    - The function does not return a value; it performs the side effect of drawing the status line on the terminal screen.


---
### screen_redraw_draw_pane<!-- {{#callable:screen_redraw_draw_pane}} -->
The `screen_redraw_draw_pane` function is responsible for rendering a specific window pane on the terminal screen, taking into account its visibility and position within the overall screen context.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure that contains the context for the screen redraw operation, including the client, offsets, and dimensions of the visible area.
    - `wp`: A pointer to a `window_pane` structure representing the pane to be drawn, including its position, size, and associated screen and palette.
- **Control Flow**:
    - Check if the pane is within the visible area of the screen using its offsets and dimensions; if not, return immediately.
    - Determine the top offset based on whether the status line is at the top or bottom of the screen.
    - Iterate over each line of the pane's height (`sy`), checking if the line is within the visible vertical range of the screen; if not, skip to the next line.
    - For each visible line, calculate the starting position (`x`) and width of the line to be drawn based on the pane's horizontal position and the screen's visible range.
    - Log the details of the line being drawn for debugging purposes.
    - Set default colors for the terminal and draw the line using `tty_draw_line`, passing the terminal, screen, starting position, width, and color information.
    - If the SIXEL feature is enabled, draw images associated with the pane using `tty_draw_images`.
- **Output**:
    - The function does not return a value; it performs drawing operations directly on the terminal associated with the client in the context.


---
### screen_redraw_draw_pane_scrollbars<!-- {{#callable:screen_redraw_draw_pane_scrollbars}} -->
The function `screen_redraw_draw_pane_scrollbars` iterates over all window panes in the current window and draws scrollbars for those that are visible and have scrollbars enabled.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains context information for the screen redraw operation, including the client and pane scrollbar settings.
- **Control Flow**:
    - Retrieve the current client and window from the context structure.
    - Log a debug message with the function name, client name, and window ID.
    - Iterate over each window pane in the current window using a TAILQ_FOREACH loop.
    - For each pane, check if the pane should show a scrollbar and if it is visible using `window_pane_show_scrollbar` and `window_pane_visible`.
    - If both conditions are true, call [`screen_redraw_draw_pane_scrollbar`](#screen_redraw_draw_pane_scrollbar) to draw the scrollbar for the pane.
- **Output**:
    - This function does not return a value; it performs an action by drawing scrollbars on the screen for eligible panes.
- **Functions called**:
    - [`screen_redraw_draw_pane_scrollbar`](#screen_redraw_draw_pane_scrollbar)


---
### screen_redraw_draw_pane_scrollbar<!-- {{#callable:screen_redraw_draw_pane_scrollbar}} -->
The function `screen_redraw_draw_pane_scrollbar` calculates and draws the scrollbar for a given window pane based on its current mode and position.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure that contains context information for the screen redraw operation, including scrollbar settings and offsets.
    - `wp`: A pointer to a `window_pane` structure representing the pane for which the scrollbar is being drawn, containing information about the pane's screen, size, and scrollbar style.
- **Control Flow**:
    - Retrieve the screen associated with the window pane `wp` and initialize variables for scrollbar dimensions and position.
    - Check if the window pane is in a mode that does not require a scrollbar; if so, return early if the scrollbar is modal.
    - Calculate the total height of the content and the percentage of the viewable area to determine the slider's height and position.
    - Adjust the slider's position and height based on the current mode and content offset if the pane is in a copy mode.
    - Determine the x-coordinate of the scrollbar based on its position (left or right of the pane).
    - Ensure the slider's height is at least 1 and adjust its position if it exceeds the scrollbar height.
    - Call [`screen_redraw_draw_scrollbar`](#screen_redraw_draw_scrollbar) to render the scrollbar on the screen with the calculated dimensions and position.
    - Store the current position and height of the slider in the `window_pane` structure for future reference.
- **Output**:
    - The function does not return a value; it performs the side effect of drawing the scrollbar on the screen and updates the `window_pane` structure with the slider's position and height.
- **Functions called**:
    - [`screen_redraw_draw_scrollbar`](#screen_redraw_draw_scrollbar)


---
### screen_redraw_draw_scrollbar<!-- {{#callable:screen_redraw_draw_scrollbar}} -->
The function `screen_redraw_draw_scrollbar` draws a scrollbar on a window pane within a terminal screen, adjusting for the pane's position and size.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which contains context information for the screen redraw, including client and terminal details.
    - `wp`: A pointer to a `window_pane` structure, representing the pane for which the scrollbar is being drawn.
    - `sb_pos`: An integer indicating the position of the scrollbar, either left or right of the pane.
    - `sb_x`: An integer representing the x-coordinate of the scrollbar's starting position.
    - `sb_y`: An integer representing the y-coordinate of the scrollbar's starting position.
    - `sb_h`: An unsigned integer representing the height of the scrollbar.
    - `slider_h`: An unsigned integer representing the height of the slider within the scrollbar.
    - `slider_y`: An unsigned integer representing the y-coordinate of the slider's starting position within the scrollbar.
- **Control Flow**:
    - Initialize local variables and retrieve the client and terminal from the context.
    - Set up the style for the slider by swapping the foreground and background colors of the scrollbar style.
    - Calculate the maximum width (`imax`) and height (`jmax`) of the scrollbar, ensuring it fits within the terminal's dimensions.
    - Iterate over each row (`j`) and column (`i`) within the scrollbar's dimensions.
    - Calculate the absolute position (`px`, `py`) of each cell in the terminal.
    - Check if the cell is within the visible area of the terminal and skip if not.
    - Move the terminal cursor to the calculated position (`px`, `py`).
    - Determine if the current cell is part of the slider or the scrollbar background and draw the appropriate cell using `tty_cell`.
- **Output**:
    - The function does not return a value; it directly manipulates the terminal display to draw the scrollbar.


# Function Declarations (Public API)

---
### screen_redraw_draw_borders<!-- {{#callable_declaration:screen_redraw_draw_borders}} -->
Draws borders around the panes in the current window.
- **Description**: This function is used to render the borders around each pane within the current window of a client session. It should be called when the pane layout changes or when a full redraw of the screen is necessary. The function iterates over each cell in the terminal's visible area, determining if it is part of a pane border and drawing it accordingly. It requires a valid `screen_redraw_ctx` structure, which must be properly initialized with the current client and session context before calling this function.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure that contains the context for the redraw operation. This includes information about the client, session, and window. The pointer must not be null, and the structure must be initialized before use.
- **Output**: None
- **See also**: [`screen_redraw_draw_borders`](#screen_redraw_draw_borders)  (Implementation)


---
### screen_redraw_draw_panes<!-- {{#callable_declaration:screen_redraw_draw_panes}} -->
Redraws all visible panes in the current window.
- **Description**: Use this function to refresh the display of all visible panes within the current window of a client session. It should be called when the visual representation of panes needs updating, such as after changes to pane content or layout. This function assumes that the context has been properly set up with a valid client and window session. It iterates over each pane in the current window and redraws those that are visible, ensuring the display is up-to-date.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure, which must be initialized and contain valid references to the client and current window session. The context should not be null, and it is expected to be properly set up before calling this function.
- **Output**: None
- **See also**: [`screen_redraw_draw_panes`](#screen_redraw_draw_panes)  (Implementation)


---
### screen_redraw_draw_status<!-- {{#callable_declaration:screen_redraw_draw_status}} -->
Redraws the status line on the screen.
- **Description**: This function is used to redraw the status line on the screen for a given client context. It should be called when the status line needs to be updated, such as when the content or appearance of the status line changes. The function assumes that the client and its associated session and window are properly initialized and that the status line is enabled. It handles drawing the status line at the top or bottom of the screen based on the context configuration.
- **Inputs**:
    - `ctx`: A pointer to a 'struct screen_redraw_ctx' which contains the context for the screen redraw operation. This includes information about the client, the status line position, and other relevant settings. Must not be null.
- **Output**: None
- **See also**: [`screen_redraw_draw_status`](#screen_redraw_draw_status)  (Implementation)


---
### screen_redraw_draw_pane<!-- {{#callable_declaration:screen_redraw_draw_pane}} -->
Redraws a specific window pane on the screen.
- **Description**: This function is used to redraw a specific window pane within the context of a screen redraw operation. It should be called when a particular pane needs to be updated, such as when its contents have changed or when the pane needs to be refreshed due to a change in visibility or position. The function takes into account the current offset and size of the pane relative to the screen, ensuring that only the visible portions are redrawn. It is important to ensure that the context and window pane provided are valid and properly initialized before calling this function.
- **Inputs**:
    - `ctx`: A pointer to a screen_redraw_ctx structure that provides the context for the redraw operation. This must be a valid, non-null pointer, and the context should be properly initialized before use.
    - `wp`: A pointer to a window_pane structure representing the pane to be redrawn. This must be a valid, non-null pointer, and the pane should be visible and properly initialized.
- **Output**: None
- **See also**: [`screen_redraw_draw_pane`](#screen_redraw_draw_pane)  (Implementation)


---
### screen_redraw_set_context<!-- {{#callable_declaration:screen_redraw_set_context}} -->
Set up the screen redraw context for a client.
- **Description**: This function prepares the screen redraw context for a given client, initializing the context structure with relevant display settings and options. It should be called before any screen redraw operations to ensure that the context is correctly configured. The function assumes that the client and its associated session and window are valid and properly initialized. It does not return any value or modify the client directly, but it populates the provided context structure with necessary information for subsequent redraw operations.
- **Inputs**:
    - `c`: A pointer to a 'client' structure representing the client for which the screen redraw context is being set. Must not be null.
    - `ctx`: A pointer to a 'screen_redraw_ctx' structure that will be populated with the redraw context information. Must not be null.
- **Output**: None
- **See also**: [`screen_redraw_set_context`](#screen_redraw_set_context)  (Implementation)


---
### screen_redraw_draw_pane_scrollbars<!-- {{#callable_declaration:screen_redraw_draw_pane_scrollbars}} -->
Draws scrollbars for visible panes in the current window.
- **Description**: This function is used to draw scrollbars for each visible pane within the current window of a client session. It should be called when the screen needs to be redrawn to reflect the current state of pane scrollbars. The function iterates over all panes in the current window and draws a scrollbar for each pane that is visible and configured to show a scrollbar. This function assumes that the context has been properly set up with the necessary client and window information.
- **Inputs**:
    - `ctx`: A pointer to a `screen_redraw_ctx` structure that contains the context for the screen redraw operation. This includes information about the client, the current window, and settings related to pane scrollbars. The pointer must not be null, and the context should be properly initialized before calling this function.
- **Output**: None
- **See also**: [`screen_redraw_draw_pane_scrollbars`](#screen_redraw_draw_pane_scrollbars)  (Implementation)


---
### screen_redraw_draw_scrollbar<!-- {{#callable_declaration:screen_redraw_draw_scrollbar}} -->
Draws a scrollbar for a window pane.
- **Description**: This function is used to render a scrollbar on a specified window pane within a terminal interface. It should be called when a visual representation of a scrollbar is needed, typically as part of a screen redraw operation. The function requires a context that includes the client and terminal information, as well as details about the window pane and scrollbar dimensions. It assumes that the context and window pane are properly initialized and visible. The function does not handle invalid input values, so parameters should be validated before calling.
- **Inputs**:
    - `ctx`: A pointer to a 'screen_redraw_ctx' structure that contains the context for the screen redraw, including client and terminal information. Must not be null.
    - `wp`: A pointer to a 'window_pane' structure representing the pane for which the scrollbar is being drawn. Must not be null and should be visible.
    - `sb_pos`: An integer indicating the position of the scrollbar, typically defined by constants such as PANE_SCROLLBARS_LEFT or PANE_SCROLLBARS_RIGHT.
    - `sb_x`: An integer specifying the x-coordinate of the scrollbar's starting position on the screen.
    - `sb_y`: An integer specifying the y-coordinate of the scrollbar's starting position on the screen.
    - `sb_h`: An unsigned integer representing the height of the scrollbar in terminal rows.
    - `slider_h`: An unsigned integer representing the height of the slider within the scrollbar.
    - `slider_y`: An unsigned integer indicating the y-coordinate of the slider's starting position within the scrollbar.
- **Output**: None
- **See also**: [`screen_redraw_draw_scrollbar`](#screen_redraw_draw_scrollbar)  (Implementation)


---
### screen_redraw_draw_pane_scrollbar<!-- {{#callable_declaration:screen_redraw_draw_pane_scrollbar}} -->
Draws a scrollbar for a specified window pane.
- **Description**: This function is used to render a scrollbar for a given window pane within a screen redraw context. It should be called when a pane's scrollbar needs to be updated or initially drawn. The function calculates the position and size of the scrollbar slider based on the pane's current view and mode. It is important to ensure that the context and window pane provided are valid and properly initialized before calling this function.
- **Inputs**:
    - `ctx`: A pointer to a screen_redraw_ctx structure that provides the context for the screen redraw operation. Must not be null.
    - `wp`: A pointer to a window_pane structure representing the pane for which the scrollbar is to be drawn. Must not be null and should be a visible pane.
- **Output**: None
- **See also**: [`screen_redraw_draw_pane_scrollbar`](#screen_redraw_draw_pane_scrollbar)  (Implementation)


