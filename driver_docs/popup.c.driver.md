# Purpose
The provided C source code is part of the tmux project, specifically handling the creation and management of pop-up windows within the terminal multiplexer environment. This file defines the structures and functions necessary to display, interact with, and manage pop-up windows, which can be used for various purposes such as displaying menus, running shell commands, or editing text within a temporary file. The code includes detailed implementations for drawing the pop-up, handling user input (including mouse events), resizing, and closing the pop-up. It also manages the integration of pop-ups with the tmux client, ensuring that the pop-up's lifecycle is properly handled, including cleanup and resource deallocation.

The code is structured around several key components, including the `popup_data` structure, which holds the state and configuration of a pop-up, and various callback functions that manage the pop-up's behavior in response to user actions and system events. The file also defines menu items and handles the execution of shell commands within a pop-up, using job control to manage the execution and output of these commands. Additionally, the code provides functionality for editing text within a pop-up using an external editor, leveraging temporary files to store the content being edited. This file is a crucial part of the tmux codebase, providing a flexible and interactive interface for users to interact with pop-ups in a terminal environment.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/wait.h`
- `signal.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### popup_menu_items
- **Type**: `static const struct menu_item[]`
- **Description**: The `popup_menu_items` is a static constant array of `struct menu_item` that defines a set of menu items for a popup menu. Each element in the array represents a menu item with a label, a key code, and a NULL pointer for additional data. The array is terminated with a NULL entry to indicate the end of the menu items.
- **Use**: This variable is used to define the items available in a popup menu, allowing users to interact with the menu through specific key codes.


---
### popup_internal_menu_items
- **Type**: `static const struct menu_item[]`
- **Description**: The `popup_internal_menu_items` is a static constant array of `struct menu_item` that defines a set of menu items for an internal popup menu. Each menu item consists of a label, a key code, and a NULL pointer, indicating no additional data or action is associated with the item.
- **Use**: This array is used to define the options available in an internal popup menu, providing labels and key codes for user interaction.


# Data Structures

---
### popup_data
- **Type**: `struct`
- **Members**:
    - `c`: Pointer to a client structure associated with the popup.
    - `item`: Pointer to a command queue item related to the popup.
    - `flags`: Integer representing various flags for the popup.
    - `title`: String representing the title of the popup.
    - `border_cell`: Grid cell structure defining the border appearance of the popup.
    - `border_lines`: Enumeration defining the style of border lines for the popup.
    - `s`: Screen structure representing the content area of the popup.
    - `defaults`: Grid cell structure holding default styling for the popup.
    - `palette`: Colour palette used for the popup's display.
    - `job`: Pointer to a job structure associated with the popup.
    - `ictx`: Pointer to an input context for handling input events.
    - `status`: Integer representing the status of the popup, often related to job completion.
    - `cb`: Callback function to be called when the popup is closed.
    - `arg`: Pointer to additional arguments for the callback function.
    - `menu`: Pointer to a menu structure if the popup includes a menu.
    - `md`: Pointer to menu data associated with the popup.
    - `close`: Integer flag indicating if the popup should be closed.
    - `px`: Unsigned integer representing the current x-coordinate of the popup.
    - `py`: Unsigned integer representing the current y-coordinate of the popup.
    - `sx`: Unsigned integer representing the current width of the popup.
    - `sy`: Unsigned integer representing the current height of the popup.
    - `ppx`: Unsigned integer representing the preferred x-coordinate of the popup.
    - `ppy`: Unsigned integer representing the preferred y-coordinate of the popup.
    - `psx`: Unsigned integer representing the preferred width of the popup.
    - `psy`: Unsigned integer representing the preferred height of the popup.
    - `dragging`: Enumeration indicating the current dragging state of the popup.
    - `dx`: Unsigned integer representing the x-offset during dragging.
    - `dy`: Unsigned integer representing the y-offset during dragging.
    - `lx`: Unsigned integer representing the last x-coordinate of a mouse event.
    - `ly`: Unsigned integer representing the last y-coordinate of a mouse event.
    - `lb`: Unsigned integer representing the last button state of a mouse event.
- **Description**: The `popup_data` structure is a comprehensive data structure used to manage the state and behavior of a popup window within a terminal multiplexer environment. It includes pointers to client and command queue items, styling information such as border and default cell styles, and a color palette for rendering. The structure also manages job execution and input handling through associated pointers and status indicators. Additionally, it maintains current and preferred dimensions and positions, allowing for dynamic resizing and movement, and supports menu integration and callback functionality for closing events.


---
### popup_editor
- **Type**: `struct`
- **Members**:
    - `path`: A pointer to a character array representing the file path.
    - `cb`: A callback function of type `popup_finish_edit_cb` to be called upon completion of editing.
    - `arg`: A void pointer to user-defined data to be passed to the callback function.
- **Description**: The `popup_editor` structure is designed to manage the state and behavior of a popup editor in a terminal-based application. It holds a file path to the temporary file being edited, a callback function to handle the completion of the editing process, and a generic argument pointer for passing additional data to the callback. This structure facilitates the integration of an external editor within the application, allowing for temporary file editing and subsequent processing of the edited content.


# Functions

---
### popup_redraw_cb<!-- {{#callable:popup_redraw_cb}} -->
The `popup_redraw_cb` function sets a flag to indicate that a client needs to redraw its overlay.
- **Inputs**:
    - `ttyctx`: A pointer to a `tty_ctx` structure, which contains context information for terminal operations, including a pointer to a `popup_data` structure as its argument.
- **Control Flow**:
    - Retrieve the `popup_data` structure from the `ttyctx` argument.
    - Set the `CLIENT_REDRAWOVERLAY` flag in the `flags` field of the `client` structure within `popup_data`.
- **Output**:
    - The function does not return any value; it modifies the `flags` field of the `client` structure to indicate that the overlay should be redrawn.


---
### popup_set_client_cb<!-- {{#callable:popup_set_client_cb}} -->
The `popup_set_client_cb` function configures the terminal context for a popup display based on the client and popup data.
- **Inputs**:
    - `ttyctx`: A pointer to a `tty_ctx` structure that holds terminal context information.
    - `c`: A pointer to a `client` structure representing the client for which the popup is being set.
- **Control Flow**:
    - Retrieve the `popup_data` structure from the `ttyctx` argument.
    - Check if the client `c` is the same as the client in `popup_data`; if not, return 0.
    - Check if the client has the `CLIENT_REDRAWOVERLAY` flag set; if so, return 0.
    - Set `ttyctx->bigger`, `ttyctx->wox`, and `ttyctx->woy` to 0.
    - Set `ttyctx->wsx` and `ttyctx->wsy` to the client's terminal size (`c->tty.sx` and `c->tty.sy`).
    - If `border_lines` in `popup_data` is `BOX_LINES_NONE`, set `ttyctx->xoff` and `ttyctx->yoff` to `pd->px` and `pd->py`, respectively.
    - Otherwise, set `ttyctx->xoff` and `ttyctx->yoff` to `pd->px + 1` and `pd->py + 1`, respectively.
    - Return 1 to indicate successful configuration.
- **Output**:
    - Returns an integer value: 0 if the client does not match or if the client has the `CLIENT_REDRAWOVERLAY` flag set, and 1 if the terminal context is successfully configured.


---
### popup_init_ctx_cb<!-- {{#callable:popup_init_ctx_cb}} -->
The `popup_init_ctx_cb` function initializes a `tty_ctx` structure with default values and callbacks from a `popup_data` structure associated with a `screen_write_ctx`.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure, which contains the context for screen writing operations and includes a pointer to a `popup_data` structure as its argument.
    - `ttyctx`: A pointer to a `tty_ctx` structure, which is used to store terminal context information and will be initialized by this function.
- **Control Flow**:
    - Retrieve the `popup_data` structure from the `arg` field of the `ctx` parameter.
    - Copy the `defaults` field from the `popup_data` structure to the `defaults` field of the `ttyctx` structure.
    - Set the `palette` field of the `ttyctx` structure to point to the `palette` field of the `popup_data` structure.
    - Assign the `popup_redraw_cb` function to the `redraw_cb` field of the `ttyctx` structure.
    - Assign the `popup_set_client_cb` function to the `set_client_cb` field of the `ttyctx` structure.
    - Set the `arg` field of the `ttyctx` structure to point to the `popup_data` structure.
- **Output**:
    - The function does not return a value; it modifies the `ttyctx` structure in place.


---
### popup_mode_cb<!-- {{#callable:popup_mode_cb}} -->
The `popup_mode_cb` function calculates the cursor position for a popup window and returns the associated screen structure.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `data`: A pointer to a `popup_data` structure containing information about the popup.
    - `cx`: A pointer to an unsigned integer where the calculated x-coordinate of the cursor will be stored.
    - `cy`: A pointer to an unsigned integer where the calculated y-coordinate of the cursor will be stored.
- **Control Flow**:
    - Check if the `md` (menu data) field in the `popup_data` structure is not NULL.
    - If `md` is not NULL, call [`menu_mode_cb`](menu.c.driver.md#menu_mode_cb) with the client, menu data, and cursor coordinates, and return its result.
    - If `md` is NULL, check if the `border_lines` field in `popup_data` is `BOX_LINES_NONE`.
    - If `border_lines` is `BOX_LINES_NONE`, set `*cx` to `px + s.cx` and `*cy` to `py + s.cy`.
    - If `border_lines` is not `BOX_LINES_NONE`, set `*cx` to `px + 1 + s.cx` and `*cy` to `py + 1 + s.cy`.
    - Return a pointer to the `screen` structure within `popup_data`.
- **Output**:
    - A pointer to the `screen` structure associated with the popup.
- **Functions called**:
    - [`menu_mode_cb`](menu.c.driver.md#menu_mode_cb)


---
### popup_check_cb<!-- {{#callable:popup_check_cb}} -->
The `popup_check_cb` function determines which parts of an input range are not obstructed by a popup and updates the overlay ranges accordingly.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the popup is being checked.
    - `data`: A pointer to a `popup_data` structure containing information about the popup, including its position and size.
    - `px`: The x-coordinate of the starting point of the input range to be checked.
    - `py`: The y-coordinate of the starting point of the input range to be checked.
    - `nx`: The width of the input range to be checked.
    - `r`: A pointer to an `overlay_ranges` structure where the unobstructed parts of the input range will be stored.
- **Control Flow**:
    - Check if the `popup_data` structure has a non-null `md` (menu data) field.
    - If `md` is not null, call [`menu_check_cb`](menu.c.driver.md#menu_check_cb) to check the menu against the popup and update the overlay ranges.
    - Iterate over two possible ranges and call [`server_client_overlay_range`](server-client.c.driver.md#server_client_overlay_range) to determine unobstructed parts of the input range for each.
    - Collect non-overlapping unobstructed ranges into the output `overlay_ranges` structure.
    - Zero out any remaining ranges in the `overlay_ranges` structure if they are not used.
    - If `md` is null, directly call [`server_client_overlay_range`](server-client.c.driver.md#server_client_overlay_range) to determine unobstructed parts of the input range without menu considerations.
- **Output**:
    - The function updates the `overlay_ranges` structure `r` with the parts of the input range that are not obstructed by the popup.
- **Functions called**:
    - [`menu_check_cb`](menu.c.driver.md#menu_check_cb)
    - [`server_client_overlay_range`](server-client.c.driver.md#server_client_overlay_range)


---
### popup_draw_cb<!-- {{#callable:popup_draw_cb}} -->
The `popup_draw_cb` function is responsible for rendering a popup on the screen, including its border, title, and content, and managing overlay checks for the client.
- **Inputs**:
    - `c`: A pointer to the client structure representing the client for which the popup is being drawn.
    - `data`: A pointer to the popup_data structure containing information about the popup, such as its position, size, border, and content.
    - `rctx`: A pointer to the screen_redraw_ctx structure used for managing the screen redraw context.
- **Control Flow**:
    - Initialize a screen structure `s` with the dimensions specified in `popup_data` and start a screen write context `ctx` for it.
    - Clear the screen using [`screen_write_clearscreen`](screen-write.c.driver.md#screen_write_clearscreen).
    - Check if the popup has no border lines; if so, copy the content directly to the screen, otherwise draw a border box and then copy the content inside the border.
    - Stop the screen write context after drawing the content.
    - Adjust the default foreground and background colors using the color palette if they are set to 8 (default).
    - Set the client's overlay check and data to handle menu drawing if `md` (menu data) is present, otherwise set them to NULL.
    - Iterate over each line of the popup's height and draw it on the terminal using [`tty_draw_line`](tty.c.driver.md#tty_draw_line).
    - Free the screen resources after drawing.
    - If menu data is present, reset the overlay check and data, and call [`menu_draw_cb`](menu.c.driver.md#menu_draw_cb) to draw the menu.
    - Finally, set the client's overlay check and data to handle popup checks.
- **Output**:
    - The function does not return a value; it performs operations to draw the popup on the client's terminal and manage overlay checks.
- **Functions called**:
    - [`screen_init`](screen.c.driver.md#screen_init)
    - [`screen_write_start`](screen-write.c.driver.md#screen_write_start)
    - [`screen_write_clearscreen`](screen-write.c.driver.md#screen_write_clearscreen)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`screen_write_fast_copy`](screen-write.c.driver.md#screen_write_fast_copy)
    - [`screen_write_box`](screen-write.c.driver.md#screen_write_box)
    - [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop)
    - [`tty_draw_line`](tty.c.driver.md#tty_draw_line)
    - [`screen_free`](screen.c.driver.md#screen_free)
    - [`menu_draw_cb`](menu.c.driver.md#menu_draw_cb)


---
### popup_free_cb<!-- {{#callable:popup_free_cb}} -->
The `popup_free_cb` function is responsible for cleaning up and freeing resources associated with a popup, including executing any callback functions and freeing memory.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client associated with the popup.
    - `data`: A pointer to a `struct popup_data` which contains all the data and resources related to the popup that need to be freed.
- **Control Flow**:
    - Retrieve the `popup_data` structure from the `data` pointer.
    - Check if the `md` (menu data) is not NULL, and if so, call [`menu_free_cb`](menu.c.driver.md#menu_free_cb) to free the menu resources.
    - If a callback (`cb`) is defined, execute it with the status and argument from `popup_data`.
    - If the `cmdq_item` is not NULL, check if its client has no session and set its return value to the popup's status, then continue the command queue item.
    - Unreference the client associated with the popup using [`server_client_unref`](server-client.c.driver.md#server_client_unref).
    - If a job is associated with the popup, free it using [`job_free`](job.c.driver.md#job_free).
    - Free the input context using [`input_free`](input.c.driver.md#input_free).
    - Free the screen and color palette resources using [`screen_free`](screen.c.driver.md#screen_free) and [`colour_palette_free`](colour.c.driver.md#colour_palette_free).
    - Free the title string and the `popup_data` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup operations and frees resources.
- **Functions called**:
    - [`menu_free_cb`](menu.c.driver.md#menu_free_cb)
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue)
    - [`server_client_unref`](server-client.c.driver.md#server_client_unref)
    - [`job_free`](job.c.driver.md#job_free)
    - [`input_free`](input.c.driver.md#input_free)
    - [`screen_free`](screen.c.driver.md#screen_free)
    - [`colour_palette_free`](colour.c.driver.md#colour_palette_free)


---
### popup_resize_cb<!-- {{#callable:popup_resize_cb}} -->
The `popup_resize_cb` function adjusts the size and position of a popup window based on the terminal's dimensions and updates the screen and job sizes accordingly.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `data`: A pointer to a `popup_data` structure containing information about the popup's current and preferred size and position, as well as other related data.
- **Control Flow**:
    - Check if the `popup_data` pointer `pd` is NULL and return if it is.
    - If `pd->md` is not NULL, call [`menu_free_cb`](menu.c.driver.md#menu_free_cb) to free the menu data associated with the popup.
    - Adjust the popup's size (`pd->sx`, `pd->sy`) to fit within the terminal's dimensions (`tty->sx`, `tty->sy`).
    - Adjust the popup's position (`pd->px`, `pd->py`) to ensure it is fully visible within the terminal's dimensions.
    - If the popup has no border lines, resize the screen to the popup's size and resize the job if it exists.
    - If the popup has border lines and is larger than 2x2, resize the screen and job to account for the border.
- **Output**:
    - The function does not return a value; it modifies the `popup_data` structure in place to update the popup's size and position.
- **Functions called**:
    - [`menu_free_cb`](menu.c.driver.md#menu_free_cb)
    - [`screen_resize`](screen.c.driver.md#screen_resize)
    - [`job_resize`](job.c.driver.md#job_resize)


---
### popup_make_pane<!-- {{#callable:popup_make_pane}} -->
The `popup_make_pane` function creates a new pane in a window, potentially transferring a job to it, and sets it as the active pane.
- **Inputs**:
    - `pd`: A pointer to a `popup_data` structure containing information about the popup and its associated client, session, and job.
    - `type`: An enum value of `layout_type` that specifies the type of layout for splitting the pane.
- **Control Flow**:
    - Retrieve the client, session, and current window from the `popup_data` structure.
    - Unzoom the current window to ensure it is not in a zoomed state.
    - Split the active pane in the window using the specified layout type and create a new layout cell.
    - Retrieve the history limit from the session options and add a new pane to the window with this limit.
    - Assign the new pane to the layout cell created earlier.
    - If a job is associated with the popup, transfer it to the new pane and clear the job from the popup data.
    - Set the title of the new pane's screen and copy the popup's screen data to the new pane's base screen.
    - Resize the new pane's screen to match its dimensions and reinitialize the popup's screen to a minimal size.
    - Retrieve the default shell from the session options, validate it, and assign it to the new pane.
    - Set the new pane as the active pane in the window and mark it as changed.
    - Set the `close` flag in the popup data to indicate the popup should be closed.
- **Output**:
    - The function does not return a value; it modifies the state of the window and pane structures, and potentially the popup data.
- **Functions called**:
    - [`window_unzoom`](window.c.driver.md#window_unzoom)
    - [`layout_split_pane`](layout.c.driver.md#layout_split_pane)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`window_add_pane`](window.c.driver.md#window_add_pane)
    - [`layout_assign_pane`](layout.c.driver.md#layout_assign_pane)
    - [`job_transfer`](job.c.driver.md#job_transfer)
    - [`screen_set_title`](screen.c.driver.md#screen_set_title)
    - [`screen_free`](screen.c.driver.md#screen_free)
    - [`screen_resize`](screen.c.driver.md#screen_resize)
    - [`screen_init`](screen.c.driver.md#screen_init)
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`checkshell`](tmux.c.driver.md#checkshell)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`window_pane_set_event`](window.c.driver.md#window_pane_set_event)
    - [`window_set_active_pane`](window.c.driver.md#window_set_active_pane)


---
### popup_menu_done<!-- {{#callable:popup_menu_done}} -->
The `popup_menu_done` function handles the completion of a popup menu interaction by performing actions based on the key pressed.
- **Inputs**:
    - `menu`: A pointer to a `menu` structure, which is unused in this function.
    - `choice`: An unsigned integer representing the menu choice, which is unused in this function.
    - `key`: A `key_code` representing the key pressed by the user to select a menu option.
    - `data`: A pointer to user-defined data, specifically a `popup_data` structure, which contains information about the popup and its associated client.
- **Control Flow**:
    - The function begins by casting the `data` pointer to a `popup_data` structure and retrieves the associated client.
    - It sets the `md` and `menu` fields of the `popup_data` structure to `NULL` and calls [`server_redraw_client`](server-fn.c.driver.md#server_redraw_client) to refresh the client's display.
    - A switch statement is used to determine the action based on the `key` value.
    - If the key is 'p', it retrieves the top paste buffer and writes its data to the job's event buffer if available.
    - If the key is 'F', it sets the popup's size to fill the client's terminal and redraws the client.
    - If the key is 'C', it centers the popup within the client's terminal and redraws the client.
    - If the key is 'h', it calls [`popup_make_pane`](#popup_make_pane) to convert the popup into a horizontal pane.
    - If the key is 'v', it calls [`popup_make_pane`](#popup_make_pane) to convert the popup into a vertical pane.
    - If the key is 'q', it sets the `close` field of the `popup_data` structure to 1, indicating the popup should be closed.
- **Output**:
    - The function does not return a value; it performs actions based on the key pressed to modify the popup's state or appearance.
- **Functions called**:
    - [`server_redraw_client`](server-fn.c.driver.md#server_redraw_client)
    - [`paste_get_top`](paste.c.driver.md#paste_get_top)
    - [`paste_buffer_data`](paste.c.driver.md#paste_buffer_data)
    - [`job_get_event`](job.c.driver.md#job_get_event)
    - [`popup_make_pane`](#popup_make_pane)


---
### popup_handle_drag<!-- {{#callable:popup_handle_drag}} -->
The `popup_handle_drag` function manages the dragging behavior of a popup window, allowing it to be moved or resized based on mouse events.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client associated with the popup.
    - `pd`: A pointer to a `popup_data` structure containing information about the popup, including its position, size, and dragging state.
    - `m`: A pointer to a `mouse_event` structure representing the current mouse event, including its position and button state.
- **Control Flow**:
    - Check if the mouse event is a drag event using `MOUSE_DRAG(m->b)`.
    - If not a drag event, set `pd->dragging` to `OFF`.
    - If `pd->dragging` is `MOVE`, calculate the new position `px` and `py` for the popup based on the mouse position and constraints of the client's terminal size.
    - Update the popup's position (`pd->px`, `pd->py`) and adjust the drag offsets (`pd->dx`, `pd->dy`).
    - Redraw the client using `server_redraw_client(c)` if the popup is moved.
    - If `pd->dragging` is `SIZE`, check if the mouse position is within valid resizing bounds based on `pd->border_lines`.
    - Calculate the new size `sx` and `sy` for the popup and update `pd->sx`, `pd->sy`, `pd->psx`, and `pd->psy`.
    - Resize the popup's screen and job (if any) using [`screen_resize`](screen.c.driver.md#screen_resize) and [`job_resize`](job.c.driver.md#job_resize) functions.
    - Redraw the client using `server_redraw_client(c)` if the popup is resized.
- **Output**:
    - The function does not return a value; it modifies the `popup_data` structure to update the popup's position and size, and triggers a redraw of the client.
- **Functions called**:
    - [`server_redraw_client`](server-fn.c.driver.md#server_redraw_client)
    - [`screen_resize`](screen.c.driver.md#screen_resize)
    - [`job_resize`](job.c.driver.md#job_resize)


---
### popup_key_cb<!-- {{#callable:popup_key_cb}} -->
The `popup_key_cb` function handles key and mouse events for a popup interface, managing interactions such as dragging, resizing, and menu activation.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client interacting with the popup.
    - `data`: A pointer to the `popup_data` structure containing the state and configuration of the popup.
    - `event`: A pointer to the `key_event` structure representing the key or mouse event to be processed.
- **Control Flow**:
    - Check if a menu is active (`pd->md != NULL`), and if so, delegate the event to [`menu_key_cb`](menu.c.driver.md#menu_key_cb) and handle menu closure or redraw if necessary.
    - If the event is a mouse event (`KEYC_IS_MOUSE(event->key)`), handle dragging if active, check if the mouse is outside the popup, and potentially activate a menu if the right mouse button is clicked on a border.
    - If the event is a key event, check for specific keys (e.g., escape or Ctrl+C) to close the popup if certain flags are not set.
    - If a job is associated with the popup (`pd->job != NULL`), handle mouse events to get mouse input and write it to the job's event buffer, or process the key event directly.
    - If a menu is to be activated, create and prepare the menu, and set the client to redraw the overlay.
    - Update the last known mouse position and button state before returning.
- **Output**:
    - Returns 0 to indicate the event was handled, or 1 if the popup should be closed based on the event.
- **Functions called**:
    - [`menu_key_cb`](menu.c.driver.md#menu_key_cb)
    - [`server_client_clear_overlay`](server-client.c.driver.md#server_client_clear_overlay)
    - [`server_redraw_client`](server-fn.c.driver.md#server_redraw_client)
    - [`popup_handle_drag`](#popup_handle_drag)
    - [`input_key_get_mouse`](input-keys.c.driver.md#input_key_get_mouse)
    - [`job_get_event`](job.c.driver.md#job_get_event)
    - [`input_key`](input-keys.c.driver.md#input_key)
    - [`menu_create`](menu.c.driver.md#menu_create)
    - [`menu_add_items`](menu.c.driver.md#menu_add_items)
    - [`menu_prepare`](menu.c.driver.md#menu_prepare)


---
### popup_job_update_cb<!-- {{#callable:popup_job_update_cb}} -->
The `popup_job_update_cb` function processes data from a job's event buffer and updates the associated popup display accordingly.
- **Inputs**:
    - `job`: A pointer to a `struct job` which contains the job data and event buffer to be processed.
- **Control Flow**:
    - Retrieve the `popup_data` structure associated with the job using `job_get_data(job)`.
    - Get the event buffer from the job's event using `job_get_event(job)->input`.
    - Check if the size of the event buffer is zero; if so, return immediately as there is no data to process.
    - If `pd->md` is not NULL, set the client's overlay check and data to `menu_check_cb` and `pd->md`, respectively; otherwise, set them to NULL.
    - Call [`input_parse_screen`](input.c.driver.md#input_parse_screen) to parse the screen input using the data from the event buffer.
    - Set the client's overlay check and data to `popup_check_cb` and `pd`, respectively, to update the popup display.
    - Drain the event buffer using `evbuffer_drain(evb, size)` to remove the processed data.
- **Output**:
    - The function does not return a value; it updates the popup display and client overlay settings based on the processed job data.
- **Functions called**:
    - [`job_get_data`](job.c.driver.md#job_get_data)
    - [`job_get_event`](job.c.driver.md#job_get_event)
    - [`input_parse_screen`](input.c.driver.md#input_parse_screen)


---
### popup_job_complete_cb<!-- {{#callable:popup_job_complete_cb}} -->
The `popup_job_complete_cb` function handles the completion of a job associated with a popup, updating the popup's status and potentially clearing the overlay if certain conditions are met.
- **Inputs**:
    - `job`: A pointer to a `struct job` representing the job that has completed.
- **Control Flow**:
    - Retrieve the `popup_data` associated with the job using [`job_get_data`](job.c.driver.md#job_get_data).
    - Get the status of the job using [`job_get_status`](job.c.driver.md#job_get_status).
    - Check if the job exited normally using `WIFEXITED`; if so, set `pd->status` to the exit status using `WEXITSTATUS`.
    - If the job was terminated by a signal using `WIFSIGNALED`, set `pd->status` to the signal number using `WTERMSIG`.
    - If neither condition is met, set `pd->status` to 0.
    - Set `pd->job` to NULL, indicating the job is no longer active.
    - Check if the `POPUP_CLOSEEXIT` flag is set or if the `POPUP_CLOSEEXITZERO` flag is set and the status is 0; if either condition is true, call [`server_client_clear_overlay`](server-client.c.driver.md#server_client_clear_overlay) to clear the overlay.
- **Output**:
    - The function does not return a value; it updates the status of the popup and may clear the overlay based on the job's completion status and flags.
- **Functions called**:
    - [`job_get_data`](job.c.driver.md#job_get_data)
    - [`job_get_status`](job.c.driver.md#job_get_status)
    - [`server_client_clear_overlay`](server-client.c.driver.md#server_client_clear_overlay)


---
### popup_display<!-- {{#callable:popup_display}} -->
The `popup_display` function creates and displays a popup window in a terminal, executing a shell command within it and managing its lifecycle.
- **Inputs**:
    - `flags`: An integer representing various flags that control the behavior of the popup.
    - `lines`: An enum value indicating the type of border lines for the popup.
    - `item`: A pointer to a `cmdq_item` structure, representing a command queue item associated with the popup.
    - `px`: An unsigned integer specifying the x-coordinate of the popup's position.
    - `py`: An unsigned integer specifying the y-coordinate of the popup's position.
    - `sx`: An unsigned integer specifying the width of the popup.
    - `sy`: An unsigned integer specifying the height of the popup.
    - `env`: A pointer to an `environ` structure, representing the environment variables for the shell command.
    - `shellcmd`: A string containing the shell command to be executed in the popup.
    - `argc`: An integer representing the number of arguments for the shell command.
    - `argv`: An array of strings representing the arguments for the shell command.
    - `cwd`: A string representing the current working directory for the shell command.
    - `title`: A string representing the title of the popup window.
    - `c`: A pointer to a `client` structure, representing the client for which the popup is displayed.
    - `s`: A pointer to a `session` structure, representing the session associated with the popup.
    - `style`: A string representing the style to be applied to the popup.
    - `border_style`: A string representing the style to be applied to the popup's border.
    - `cb`: A callback function to be called when the popup is closed.
    - `arg`: A pointer to user-defined data to be passed to the callback function.
- **Control Flow**:
    - Determine the options to use based on the session or client context.
    - Adjust the border lines and dimensions based on the input parameters and options.
    - Check if the popup can fit within the client's terminal dimensions; return -1 if it cannot.
    - Allocate and initialize a `popup_data` structure to hold the popup's state and configuration.
    - Set up the popup's border and default styles, applying any specified styles.
    - Initialize the screen and color palette for the popup.
    - Set the popup's position and size attributes.
    - Run the specified shell command in a job, associating it with the popup.
    - Initialize input handling for the job's output.
    - Set the client to use the popup as an overlay, with various callbacks for handling popup events.
    - Return 0 to indicate successful creation and display of the popup.
- **Output**:
    - Returns 0 on successful creation and display of the popup, or -1 if the popup cannot be displayed due to size constraints.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`style_apply`](style.c.driver.md#style_apply)
    - [`style_set`](style.c.driver.md#style_set)
    - [`style_parse`](style.c.driver.md#style_parse)
    - [`screen_init`](screen.c.driver.md#screen_init)
    - [`screen_set_default_cursor`](screen.c.driver.md#screen_set_default_cursor)
    - [`colour_palette_init`](colour.c.driver.md#colour_palette_init)
    - [`colour_palette_from_option`](colour.c.driver.md#colour_palette_from_option)
    - [`job_run`](job.c.driver.md#job_run)
    - [`input_init`](input.c.driver.md#input_init)
    - [`job_get_event`](job.c.driver.md#job_get_event)
    - [`server_client_set_overlay`](server-client.c.driver.md#server_client_set_overlay)


---
### popup_editor_free<!-- {{#callable:popup_editor_free}} -->
The `popup_editor_free` function deallocates memory and resources associated with a `popup_editor` structure.
- **Inputs**:
    - `pe`: A pointer to a `popup_editor` structure that contains the path to be unlinked and freed, and the structure itself to be freed.
- **Control Flow**:
    - The function first calls `unlink` on the `path` member of the `popup_editor` structure to remove the file associated with the path.
    - It then frees the memory allocated for the `path` string using `free`.
    - Finally, it frees the memory allocated for the `popup_editor` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided `popup_editor` structure.


---
### popup_editor_close_cb<!-- {{#callable:popup_editor_close_cb}} -->
The `popup_editor_close_cb` function handles the closing of a popup editor by reading the file content and invoking a callback with the content or an error.
- **Inputs**:
    - `status`: An integer representing the status of the editor process; a non-zero value indicates an error.
    - `arg`: A pointer to a `popup_editor` structure, which contains information about the editor session, including the file path and callback function.
- **Control Flow**:
    - Check if the status is non-zero, indicating an error, and if so, call the callback with NULL and free the editor resources.
    - Open the file at the path specified in the `popup_editor` structure for reading.
    - If the file is successfully opened, seek to the end to determine its length, then seek back to the start.
    - Allocate a buffer of the determined length and read the file content into it.
    - If any step fails (e.g., file open, memory allocation, or read), free the buffer and set its length to zero.
    - Close the file after reading.
    - Invoke the callback function with the buffer and its length, transferring ownership of the buffer to the callback.
    - Free the `popup_editor` structure resources.
- **Output**:
    - The function does not return a value, but it calls a callback function with the file content and its length, or NULL and zero length in case of an error.
- **Functions called**:
    - [`popup_editor_free`](#popup_editor_free)


---
### popup_editor<!-- {{#callable:popup_editor}} -->
The `popup_editor` function creates a temporary file with the provided buffer content, opens it in a specified editor within a popup, and executes a callback upon completion.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the popup editor is being created.
    - `buf`: A constant character pointer to the buffer containing the initial content to be edited.
    - `len`: The size of the buffer `buf`.
    - `cb`: A callback function of type `popup_finish_edit_cb` to be called when the editing is finished.
    - `arg`: A void pointer to additional arguments to be passed to the callback function.
- **Control Flow**:
    - Retrieve the editor command from global options; if it's empty, return -1.
    - Create a temporary file using `mkstemp` and write the buffer content to it; if any step fails, return -1.
    - Allocate and initialize a `popup_editor` structure to store the temporary file path, callback, and argument.
    - Calculate the size and position of the popup based on the client's terminal size.
    - Format the command to open the editor with the temporary file path.
    - Call [`popup_display`](#popup_display) to show the editor in a popup; if it fails, free resources and return -1.
    - Free the command string and return 0 on success.
- **Output**:
    - Returns 0 on success, or -1 if an error occurs during setup or execution.
- **Functions called**:
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`popup_display`](#popup_display)
    - [`popup_editor_free`](#popup_editor_free)


