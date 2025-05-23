# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to switch between several programs in one terminal. The file is responsible for managing pop-up windows within the tmux environment. It provides functionality to create, display, and manage pop-up windows, which can be used for various purposes such as displaying menus, running shell commands, or editing text. The code defines structures and functions to handle the lifecycle of pop-ups, including their creation, rendering, resizing, and interaction with user inputs like keyboard and mouse events.

The file includes several key components: the `popup_data` structure, which holds the state and properties of a pop-up; functions for drawing and updating the pop-up display; and callback functions for handling user interactions and job completions. It also defines menu items and manages the integration of pop-ups with the tmux client interface. The code is designed to be integrated into the tmux application, providing a modular way to extend its functionality with interactive pop-up windows. The file does not define a public API but rather serves as an internal component of the tmux system, facilitating enhanced user interaction within the terminal multiplexer environment.
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
- **Description**: The `popup_menu_items` is a static constant array of `menu_item` structures, which defines a set of menu items for a popup menu. Each `menu_item` consists of a label, a key code, and a NULL pointer, which likely represents an action or callback associated with the menu item.
- **Use**: This variable is used to define the options available in a popup menu, allowing users to interact with the application through predefined commands.


---
### popup_internal_menu_items
- **Type**: ``static const struct menu_item[]``
- **Description**: The `popup_internal_menu_items` is a static constant array of `struct menu_item` that defines a set of menu items for an internal popup menu. Each element in the array represents a menu item with a label, a key code, and a NULL pointer for additional data. The array is terminated with a NULL entry to indicate the end of the menu items.
- **Use**: This variable is used to define the menu items available in an internal popup menu, providing options such as 'Close', 'Fill Space', and 'Centre'.


# Data Structures

---
### popup_data
- **Type**: `struct`
- **Members**:
    - `c`: Pointer to a client structure associated with the popup.
    - `item`: Pointer to a command queue item associated with the popup.
    - `flags`: Integer representing various flags for the popup.
    - `title`: String representing the title of the popup.
    - `border_cell`: Grid cell structure defining the border appearance of the popup.
    - `border_lines`: Enumeration defining the style of the border lines.
    - `s`: Screen structure representing the content of the popup.
    - `defaults`: Grid cell structure defining default appearance settings.
    - `palette`: Colour palette structure for the popup.
    - `job`: Pointer to a job structure associated with the popup.
    - `ictx`: Pointer to an input context structure for handling input.
    - `status`: Integer representing the status of the popup.
    - `cb`: Callback function to be called when the popup is closed.
    - `arg`: Pointer to additional arguments for the callback function.
    - `menu`: Pointer to a menu structure associated with the popup.
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
- **Description**: The `popup_data` structure is a comprehensive data structure used to manage and display popups within a terminal multiplexer environment. It encapsulates various attributes and states necessary for rendering and interacting with popups, including client associations, command queue items, visual properties like borders and colors, and input handling. The structure also manages the current and preferred positions and sizes of the popup, as well as its interaction states such as dragging. Additionally, it supports job execution and callback mechanisms for handling popup closure events.


---
### popup_editor
- **Type**: `struct`
- **Members**:
    - `path`: A pointer to a character array representing the file path.
    - `cb`: A callback function of type `popup_finish_edit_cb` to be called upon completion of editing.
    - `arg`: A void pointer to user-defined data to be passed to the callback function.
- **Description**: The `popup_editor` structure is designed to manage the state and behavior of a popup editor in a software application. It contains a file path to the temporary file being edited, a callback function to handle the completion of the editing process, and a generic argument pointer for passing additional data to the callback. This structure is used to facilitate the editing of text within a popup interface, allowing for asynchronous operations and custom handling upon completion.


# Functions

---
### popup_redraw_cb<!-- {{#callable:popup_redraw_cb}} -->
The `popup_redraw_cb` function sets a flag to indicate that a client needs to redraw its overlay.
- **Inputs**:
    - `ttyctx`: A pointer to a `tty_ctx` structure, which contains context information for terminal operations, including a pointer to `popup_data`.
- **Control Flow**:
    - Retrieve the `popup_data` structure from the `ttyctx` argument.
    - Set the `CLIENT_REDRAWOVERLAY` flag in the `flags` field of the `client` structure within `popup_data`.
- **Output**:
    - This function does not return any value; it modifies the state of the client by setting a flag.


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
    - If `border_lines` in `popup_data` is `BOX_LINES_NONE`, set `ttyctx->xoff` and `ttyctx->yoff` to `pd->px` and `pd->py`, respectively; otherwise, set them to `pd->px + 1` and `pd->py + 1`.
    - Return 1 to indicate successful configuration.
- **Output**:
    - Returns an integer value, 0 if the client does not match or if the redraw overlay flag is set, and 1 if the terminal context is successfully configured.


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
    - If `md` is not NULL, call `menu_mode_cb` with the client, menu data, and cursor coordinates, and return its result.
    - If `md` is NULL, check if `border_lines` in `popup_data` is `BOX_LINES_NONE`.
    - If `border_lines` is `BOX_LINES_NONE`, set `*cx` to `px + s.cx` and `*cy` to `py + s.cy`.
    - If `border_lines` is not `BOX_LINES_NONE`, set `*cx` to `px + 1 + s.cx` and `*cy` to `py + 1 + s.cy`.
    - Return a pointer to the `screen` structure within `popup_data`.
- **Output**:
    - A pointer to the `screen` structure associated with the popup.


---
### popup_check_cb<!-- {{#callable:popup_check_cb}} -->
The `popup_check_cb` function determines which parts of an input range are not obstructed by a popup and updates the overlay ranges accordingly.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the popup is being checked.
    - `data`: A pointer to `popup_data` structure containing information about the popup, including its position and size.
    - `px`: The x-coordinate of the starting point of the input range to be checked.
    - `py`: The y-coordinate of the starting point of the input range to be checked.
    - `nx`: The width of the input range to be checked.
    - `r`: A pointer to an `overlay_ranges` structure where the unobstructed parts of the input range will be stored.
- **Control Flow**:
    - Check if the `popup_data` structure contains a menu (`pd->md` is not NULL).
    - If a menu exists, call `menu_check_cb` to check the menu against the popup and update the overlay ranges `r`.
    - For each of the two possible overlay ranges, call `server_client_overlay_range` to determine the unobstructed parts of the range and store them in `or` array.
    - Iterate over the `or` array to collect non-zero ranges into the output `r`, updating the `px` and `nx` arrays of `r`.
    - Zero out any remaining ranges in `r` beyond the collected non-zero ranges.
    - If no menu exists, directly call `server_client_overlay_range` to determine the unobstructed parts of the input range and update `r`.
- **Output**:
    - The function updates the `overlay_ranges` structure `r` to reflect the parts of the input range that are not obstructed by the popup.


---
### popup_draw_cb<!-- {{#callable:popup_draw_cb}} -->
The `popup_draw_cb` function is responsible for rendering a popup on the screen, including its border, title, and content, and managing overlay checks for the client.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client for which the popup is being drawn.
    - `data`: A pointer to the `popup_data` structure containing information about the popup, such as its position, size, border, and content.
    - `rctx`: A pointer to the `screen_redraw_ctx` structure used for screen redraw context, though not directly used in this function.
- **Control Flow**:
    - Initialize a temporary screen `s` with the dimensions specified in `popup_data` and start a screen write context `ctx` for it.
    - Clear the screen using `screen_write_clearscreen`.
    - Check if the popup has no border lines; if so, copy the content directly to the screen, otherwise draw a border box and then copy the content inside the border.
    - Stop the screen write context after drawing the content.
    - Copy default grid cell settings from `popup_data` and adjust foreground and background colors if they are set to 8 (default).
    - Set the client's overlay check and data to handle menu drawing if `popup_data` contains menu data (`md`), otherwise set them to NULL.
    - Iterate over each line of the popup's height and draw it on the terminal using `tty_draw_line`.
    - Free the temporary screen `s` after drawing.
    - If menu data is present, reset the overlay check and data, and call `menu_draw_cb` to draw the menu.
    - Finally, set the client's overlay check and data to handle popup checks.
- **Output**:
    - The function does not return a value; it performs operations to render a popup on the client's screen.


---
### popup_free_cb<!-- {{#callable:popup_free_cb}} -->
The `popup_free_cb` function is responsible for cleaning up and freeing resources associated with a popup in a client application.
- **Inputs**:
    - `c`: A pointer to a `struct client`, representing the client associated with the popup.
    - `data`: A pointer to a `struct popup_data`, which contains all the data and resources related to the popup that need to be freed.
- **Control Flow**:
    - Retrieve the `popup_data` structure from the `data` pointer.
    - Check if `pd->md` is not NULL, and if so, call `menu_free_cb` to free the menu data.
    - Check if `pd->cb` is not NULL, and if so, call the callback function with `pd->status` and `pd->arg`.
    - If `item` is not NULL, check if the client associated with the item has no session, and if so, set its return value to `pd->status`. Then, continue the command queue item.
    - Unreference the client `pd->c` using `server_client_unref`.
    - If `pd->job` is not NULL, free the job using `job_free`.
    - Free the input context `pd->ictx` using `input_free`.
    - Free the screen `pd->s` using `screen_free`.
    - Free the color palette `pd->palette` using `colour_palette_free`.
    - Free the title string `pd->title` and the `popup_data` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup operations and frees memory.


---
### popup_resize_cb<!-- {{#callable:popup_resize_cb}} -->
The `popup_resize_cb` function adjusts the size and position of a popup window based on the terminal's dimensions and updates the screen and job sizes accordingly.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `data`: A pointer to a `popup_data` structure containing information about the popup's current and preferred size and position, as well as other related data.
- **Control Flow**:
    - Check if the `popup_data` pointer `pd` is NULL and return if it is.
    - If `pd->md` is not NULL, call `menu_free_cb` to free the menu data associated with the popup.
    - Adjust the popup's size (`pd->sx`, `pd->sy`) to fit within the terminal's dimensions (`tty->sx`, `tty->sy`).
    - Adjust the popup's position (`pd->px`, `pd->py`) to ensure it is fully visible within the terminal's dimensions.
    - If the popup has no border lines, resize the screen to the popup's size and resize the job if it exists.
    - If the popup has border lines and is large enough, resize the screen and job to account for the border.
- **Output**:
    - The function does not return a value; it modifies the `popup_data` structure in place to update the popup's size and position.


---
### popup_make_pane<!-- {{#callable:popup_make_pane}} -->
The `popup_make_pane` function creates a new pane in a window, potentially transferring a job to it, and sets it as the active pane.
- **Inputs**:
    - `pd`: A pointer to a `popup_data` structure containing information about the popup and its associated client, session, and job.
    - `type`: An enum value of type `layout_type` indicating how the pane should be split (e.g., horizontally or vertically).
- **Control Flow**:
    - Retrieve the client, session, and current window from the `popup_data` structure.
    - Unzoom the current window if it is zoomed.
    - Split the active pane in the window according to the specified layout type.
    - Retrieve the history limit from the session options.
    - Add a new pane to the window with the specified history limit.
    - Assign the new pane to the layout cell created by the split.
    - If a job is associated with the popup, transfer it to the new pane and clear the job from the popup data.
    - Set the title of the new pane's screen and copy the popup's screen data to the new pane.
    - Resize the new pane's screen to match its dimensions.
    - Initialize the popup's screen to a minimal size.
    - Retrieve the default shell from the session options and set it for the new pane, defaulting to `/bin/sh` if necessary.
    - Set the new pane as the active pane in the window and mark it as changed.
    - Set the `close` flag in the popup data to indicate that the popup should be closed.
- **Output**:
    - The function does not return a value; it modifies the state of the window and pane structures, and updates the `popup_data` structure.


---
### popup_menu_done<!-- {{#callable:popup_menu_done}} -->
The `popup_menu_done` function handles the completion of a popup menu interaction by executing actions based on the key pressed.
- **Inputs**:
    - `menu`: A pointer to the menu structure, which is unused in this function.
    - `choice`: An unsigned integer representing the menu choice, which is unused in this function.
    - `key`: A key_code representing the key pressed by the user to select a menu option.
    - `data`: A void pointer to user data, which is cast to a `popup_data` structure within the function.
- **Control Flow**:
    - The function begins by casting the `data` pointer to a `popup_data` structure and retrieves the associated client.
    - It sets the `md` and `menu` fields of the `popup_data` structure to NULL and redraws the client using `server_redraw_client`.
    - A switch statement is used to handle different actions based on the `key` value:
    - If the key is 'p', it retrieves the top paste buffer and writes its data to the job's event buffer if available.
    - If the key is 'F', it sets the popup's size to the client's terminal size and redraws the client.
    - If the key is 'C', it centers the popup within the client's terminal and redraws the client.
    - If the key is 'h', it calls [`popup_make_pane`](#popup_make_pane) to convert the popup into a horizontal pane.
    - If the key is 'v', it calls [`popup_make_pane`](#popup_make_pane) to convert the popup into a vertical pane.
    - If the key is 'q', it sets the `close` field of the `popup_data` structure to 1, indicating the popup should be closed.
- **Output**:
    - The function does not return a value; it performs actions based on the key pressed and modifies the state of the `popup_data` structure and the associated client.
- **Functions called**:
    - [`popup_make_pane`](#popup_make_pane)


---
### popup_handle_drag<!-- {{#callable:popup_handle_drag}} -->
The `popup_handle_drag` function manages the dragging behavior of a popup window, allowing it to be moved or resized based on mouse events.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client where the popup is displayed.
    - `pd`: A pointer to the `popup_data` structure containing information about the popup, including its position, size, and dragging state.
    - `m`: A pointer to the `mouse_event` structure representing the current mouse event, including its position and button state.
- **Control Flow**:
    - Check if the mouse event is a drag event using `MOUSE_DRAG(m->b)`.
    - If not a drag event, set `pd->dragging` to `OFF`.
    - If `pd->dragging` is `MOVE`, calculate the new position `px` and `py` for the popup based on the mouse position and constraints of the client's terminal size.
    - Update the popup's position (`pd->px`, `pd->py`) and the drag offset (`pd->dx`, `pd->dy`).
    - Redraw the client using `server_redraw_client(c)` if the popup is moved.
    - If `pd->dragging` is `SIZE`, check if the mouse position is within valid resizing bounds based on `pd->border_lines`.
    - Calculate the new size `sx` and `sy` for the popup based on the mouse position.
    - Resize the popup's screen and job (if any) using `screen_resize` and `job_resize`.
    - Redraw the client using `server_redraw_client(c)` if the popup is resized.
- **Output**:
    - The function does not return a value; it modifies the `popup_data` structure to update the popup's position and size, and triggers a redraw of the client.


---
### popup_key_cb<!-- {{#callable:popup_key_cb}} -->
The `popup_key_cb` function handles key and mouse events for a popup interface, managing interactions such as dragging, resizing, and menu activation.
- **Inputs**:
    - `c`: A pointer to the client structure representing the client interacting with the popup.
    - `data`: A pointer to the popup_data structure containing the state and configuration of the popup.
    - `event`: A pointer to the key_event structure representing the key or mouse event to be processed.
- **Control Flow**:
    - Check if a menu is active in the popup; if so, delegate the event to `menu_key_cb` and handle menu closure or redraw if necessary.
    - If the event is a mouse event, handle dragging if the popup is in a dragging state, or check if the mouse event is within the popup's boundaries.
    - Determine if the mouse event is on a border and handle right-click menu activation if applicable.
    - If the event is a key event, check for specific key combinations to close the popup or send input to a job if one is running.
    - If a right-click menu is triggered, create and prepare the menu, setting it up for display.
    - Update the last known mouse position and button state before returning.
- **Output**:
    - Returns 0 to indicate the event was handled, or 1 if the event should close the popup.
- **Functions called**:
    - [`popup_handle_drag`](#popup_handle_drag)


---
### popup_job_update_cb<!-- {{#callable:popup_job_update_cb}} -->
The `popup_job_update_cb` function processes data from a job's event buffer and updates the associated popup display accordingly.
- **Inputs**:
    - `job`: A pointer to a `struct job` which contains the job data and event information.
- **Control Flow**:
    - Retrieve the `popup_data` structure associated with the job using `job_get_data(job)`.
    - Get the input event buffer from the job's event using `job_get_event(job)->input`.
    - Check if the size of the event buffer is zero; if so, return immediately as there is no data to process.
    - If `popup_data` contains a menu (`pd->md` is not NULL), set the client's overlay check and data to the menu's callback and data.
    - If no menu is present, set the client's overlay check and data to NULL.
    - Parse the screen input using `input_parse_screen` with the data from the event buffer.
    - Set the client's overlay check and data to the popup's callback and data.
    - Drain the event buffer to remove the processed data.
- **Output**:
    - The function does not return a value; it updates the client's overlay settings and processes the event buffer data.


---
### popup_job_complete_cb<!-- {{#callable:popup_job_complete_cb}} -->
The `popup_job_complete_cb` function handles the completion of a job by updating the status and potentially clearing the overlay if certain conditions are met.
- **Inputs**:
    - `job`: A pointer to a `struct job` representing the job that has completed.
- **Control Flow**:
    - Retrieve the `popup_data` associated with the job using `job_get_data`.
    - Get the status of the job using `job_get_status`.
    - Check if the job exited normally using `WIFEXITED`; if true, set `pd->status` to the exit status using `WEXITSTATUS`.
    - If the job was terminated by a signal using `WIFSIGNALED`, set `pd->status` to the signal number using `WTERMSIG`.
    - If neither condition is met, set `pd->status` to 0.
    - Set `pd->job` to NULL to indicate the job is no longer active.
    - Check if the `POPUP_CLOSEEXIT` flag is set or if the `POPUP_CLOSEEXITZERO` flag is set and the status is 0; if either condition is true, call `server_client_clear_overlay` to clear the overlay.
- **Output**:
    - The function does not return a value; it updates the status of the popup and may clear the overlay based on the job's completion status and flags.


---
### popup_display<!-- {{#callable:popup_display}} -->
The `popup_display` function creates and displays a popup window in a terminal, executing a specified shell command within it.
- **Inputs**:
    - `flags`: An integer representing various flags that control the behavior of the popup.
    - `lines`: An enum value indicating the type of border lines for the popup.
    - `item`: A pointer to a `cmdq_item` structure, representing a command queue item.
    - `px`: An unsigned integer specifying the x-coordinate for the popup's position.
    - `py`: An unsigned integer specifying the y-coordinate for the popup's position.
    - `sx`: An unsigned integer specifying the width of the popup.
    - `sy`: An unsigned integer specifying the height of the popup.
    - `env`: A pointer to an `environ` structure, representing the environment variables for the shell command.
    - `shellcmd`: A string representing the shell command to be executed in the popup.
    - `argc`: An integer representing the number of arguments for the shell command.
    - `argv`: An array of strings representing the arguments for the shell command.
    - `cwd`: A string representing the current working directory for the shell command.
    - `title`: A string representing the title of the popup.
    - `c`: A pointer to a `client` structure, representing the client where the popup will be displayed.
    - `s`: A pointer to a `session` structure, representing the session associated with the popup.
    - `style`: A string representing the style to be applied to the popup.
    - `border_style`: A string representing the style to be applied to the popup's border.
    - `cb`: A callback function to be called when the popup is closed.
    - `arg`: A pointer to user-defined data to be passed to the callback function.
- **Control Flow**:
    - Determine the options to use based on whether a session is provided or not.
    - Set the border lines based on the provided `lines` argument or default options.
    - Check if the popup size is valid and return -1 if not.
    - Allocate and initialize a `popup_data` structure to store popup-related data.
    - Set the popup's title, client, callback, and other attributes.
    - Initialize the screen and color palette for the popup.
    - Run the specified shell command as a job within the popup.
    - Set the client overlay to handle popup interactions and rendering.
    - Return 0 to indicate successful popup creation.
- **Output**:
    - Returns 0 on successful creation of the popup, or -1 if there is an error with the popup size or client terminal size.


---
### popup_editor_free<!-- {{#callable:popup_editor_free}} -->
The `popup_editor_free` function deallocates memory and resources associated with a `popup_editor` structure.
- **Inputs**:
    - `pe`: A pointer to a `popup_editor` structure that contains the path to be unlinked and memory to be freed.
- **Control Flow**:
    - The function first calls `unlink` on the `path` member of the `popup_editor` structure to remove the file at that path.
    - It then frees the memory allocated for the `path` string using `free`.
    - Finally, it frees the memory allocated for the `popup_editor` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided `popup_editor` structure.


---
### popup_editor_close_cb<!-- {{#callable:popup_editor_close_cb}} -->
The `popup_editor_close_cb` function handles the closure of a popup editor by reading the contents of a file and invoking a callback with the file's data.
- **Inputs**:
    - `status`: An integer representing the status of the editor process; a non-zero value indicates an error.
    - `arg`: A pointer to a `popup_editor` structure, which contains information about the editor session, including the file path and callback function.
- **Control Flow**:
    - Check if the `status` is non-zero, indicating an error, and if so, call the callback with `NULL` and free the `popup_editor` structure.
    - Open the file specified by `pe->path` for reading.
    - If the file is successfully opened, seek to the end to determine its length, then seek back to the start.
    - Allocate a buffer of the file's length and read the file's contents into the buffer.
    - If any step fails (e.g., file length is zero, allocation fails, or read fails), free the buffer and set it to `NULL`.
    - Close the file after reading.
    - Invoke the callback with the buffer and its length, transferring ownership of the buffer to the callback.
    - Free the `popup_editor` structure.
- **Output**:
    - The function does not return a value, but it invokes a callback with the file's contents and length, or `NULL` if an error occurred.
- **Functions called**:
    - [`popup_editor_free`](#popup_editor_free)


---
### popup_editor<!-- {{#callable:popup_editor}} -->
The `popup_editor` function creates a temporary file with the provided buffer content, opens it in a specified editor within a popup, and executes a callback upon completion.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the popup editor is being created.
    - `buf`: A constant character pointer to the buffer containing the initial content to be edited.
    - `len`: A size_t value representing the length of the buffer.
    - `cb`: A callback function of type `popup_finish_edit_cb` to be called when the editing is finished.
    - `arg`: A void pointer to additional arguments to be passed to the callback function.
- **Control Flow**:
    - Retrieve the editor command from global options; if empty, return -1.
    - Create a temporary file using `mkstemp` and write the buffer content to it; handle errors by returning -1.
    - Allocate and initialize a `popup_editor` structure to store the path, callback, and argument.
    - Calculate the size and position of the popup based on the client's terminal size.
    - Format the command to open the editor with the temporary file path.
    - Call [`popup_display`](#popup_display) to show the editor in a popup; if it fails, free resources and return -1.
    - Free the command string and return 0 on success.
- **Output**:
    - Returns 0 on success, or -1 on failure.
- **Functions called**:
    - [`popup_display`](#popup_display)
    - [`popup_editor_free`](#popup_editor_free)


