# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides functionality for creating and managing interactive menus within the tmux environment. The code defines a `menu_data` structure to hold information about a menu, including its style, position, and the items it contains. The file includes functions to add items to a menu, create and free menu structures, and handle user interactions such as key presses and mouse events. The menu can be displayed on the terminal, and it supports navigation through keyboard and mouse inputs, allowing users to select options from the menu.

The code is designed to be integrated into the tmux application, as indicated by the inclusion of the "tmux.h" header file. It does not define a standalone executable but rather provides a set of functions and structures that can be used by other parts of the tmux codebase to implement menu functionality. The file includes functions for setting menu styles, handling user input, and managing the display of the menu on the terminal screen. The code is structured to support customization of menu appearance and behavior, such as border styles and selected item styles, making it a flexible component for enhancing user interaction within tmux.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Data Structures

---
### menu_data
- **Type**: `struct`
- **Members**:
    - `item`: A pointer to a command queue item associated with the menu.
    - `flags`: An integer representing various flags for menu behavior.
    - `style`: A grid cell structure defining the style of the menu.
    - `border_style`: A grid cell structure defining the style of the menu border.
    - `selected_style`: A grid cell structure defining the style of the selected menu item.
    - `border_lines`: An enumeration value representing the type of border lines used in the menu.
    - `fs`: A command find state structure used for command execution context.
    - `s`: A screen structure representing the menu's display area.
    - `px`: An unsigned integer for the x-coordinate of the menu's position.
    - `py`: An unsigned integer for the y-coordinate of the menu's position.
    - `menu`: A pointer to the menu structure being displayed.
    - `choice`: An integer representing the currently selected menu choice.
    - `cb`: A callback function for handling menu choice selection.
    - `data`: A pointer to additional data used by the callback function.
- **Description**: The `menu_data` structure is a comprehensive data structure used to manage and display a menu within a terminal-based application. It contains various members that define the menu's appearance, position, and behavior, including styles for the menu and its border, the current selection, and callback functionality for handling user interactions. The structure also includes pointers to related command queue items and menu structures, as well as coordinates for positioning the menu on the screen.


# Functions

---
### menu_add_items<!-- {{#callable:menu_add_items}} -->
The `menu_add_items` function iterates over a list of menu items and adds each one to a specified menu using the [`menu_add_item`](#menu_add_item) function.
- **Inputs**:
    - `menu`: A pointer to the `menu` structure where items will be added.
    - `items`: A pointer to an array of `menu_item` structures, each representing a menu item to be added.
    - `qitem`: A pointer to a `cmdq_item` structure, which may be used for command queue operations.
    - `c`: A pointer to a `client` structure, representing the client context.
    - `fs`: A pointer to a `cmd_find_state` structure, which may be used for command finding operations.
- **Control Flow**:
    - Initialize a loop variable `loop` to iterate over the `items` array.
    - For each `menu_item` in the `items` array, check if the `name` field is not `NULL`.
    - If the `name` is not `NULL`, call [`menu_add_item`](#menu_add_item) to add the item to the `menu`.
    - Continue the loop until a `menu_item` with a `NULL` name is encountered, indicating the end of the array.
- **Output**:
    - The function does not return a value; it modifies the `menu` structure by adding items to it.
- **Functions called**:
    - [`menu_add_item`](#menu_add_item)


---
### menu_add_item<!-- {{#callable:menu_add_item}} -->
The `menu_add_item` function adds a new item to a menu, formatting its display name and command based on the provided parameters and available space.
- **Inputs**:
    - `menu`: A pointer to the `menu` structure where the new item will be added.
    - `item`: A pointer to the `menu_item` structure containing the details of the item to be added.
    - `qitem`: A pointer to the `cmdq_item` structure used for command queue operations.
    - `c`: A pointer to the `client` structure representing the client context.
    - `fs`: A pointer to the `cmd_find_state` structure used for command state management, can be NULL.
- **Control Flow**:
    - Check if the item is a line separator or if the menu is empty, and return if true.
    - Reallocate memory for the menu items array to accommodate the new item.
    - Initialize the new item structure with zero values.
    - Format the item's name using [`format_single_from_state`](format.c.driver.md#format_single_from_state) or [`format_single`](format.c.driver.md#format_single) based on the presence of `fs`.
    - If the formatted name is empty, decrement the menu count and return.
    - Calculate the maximum width for the item based on the client's terminal size.
    - Determine if a key should be displayed with the item based on its length and available space.
    - Trim the formatted name to fit within the maximum width, appending a suffix if necessary.
    - Format the final display name of the item, including the key if applicable.
    - Assign the formatted name to the new item and free the temporary string.
    - Format the item's command if it exists, using the same logic as for the name.
    - Assign the formatted command and key to the new item.
    - Calculate the width of the new item's name and update the menu's width if necessary.
- **Output**:
    - The function does not return a value; it modifies the `menu` structure by adding a new formatted item to it.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`format_single_from_state`](format.c.driver.md#format_single_from_state)
    - [`format_single`](format.c.driver.md#format_single)
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)
    - [`format_trim_right`](format-draw.c.driver.md#format_trim_right)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`format_width`](format-draw.c.driver.md#format_width)


---
### menu_create<!-- {{#callable:menu_create}} -->
The `menu_create` function initializes and returns a new menu structure with a specified title.
- **Inputs**:
    - `title`: A constant character pointer representing the title of the menu to be created.
- **Control Flow**:
    - Allocate memory for a new menu structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc) and assign it to the `menu` pointer.
    - Duplicate the input `title` string using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and assign it to the `title` field of the `menu` structure.
    - Calculate the width of the title using [`format_width`](format-draw.c.driver.md#format_width) and assign it to the `width` field of the `menu` structure.
    - Return the pointer to the newly created `menu` structure.
- **Output**:
    - A pointer to the newly created `menu` structure, which includes the title and its width.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`format_width`](format-draw.c.driver.md#format_width)


---
### menu_free<!-- {{#callable:menu_free}} -->
The `menu_free` function deallocates all memory associated with a `menu` structure, including its items and title.
- **Inputs**:
    - `menu`: A pointer to a `struct menu` that contains the menu items, title, and other related data to be freed.
- **Control Flow**:
    - Iterate over each item in the `menu->items` array using a for loop.
    - For each item, free the memory allocated for the `name` and `command` fields.
    - After the loop, free the memory allocated for the `menu->items` array itself.
    - Free the memory allocated for the `menu->title`.
    - Finally, free the memory allocated for the `menu` structure itself.
- **Output**:
    - The function does not return any value; it performs memory deallocation for the given `menu` structure.


---
### menu_mode_cb<!-- {{#callable:menu_mode_cb}} -->
The `menu_mode_cb` function calculates the cursor position for a menu based on the menu's data and returns a pointer to the screen associated with the menu.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `data`: A pointer to a `menu_data` structure containing information about the menu, including its position and the current choice.
    - `cx`: A pointer to an unsigned integer where the function will store the calculated x-coordinate of the cursor.
    - `cy`: A pointer to an unsigned integer where the function will store the calculated y-coordinate of the cursor.
- **Control Flow**:
    - The function casts the `data` pointer to a `menu_data` structure pointer `md`.
    - It sets the x-coordinate `*cx` to `md->px + 2`.
    - It checks if `md->choice` is -1; if true, it sets the y-coordinate `*cy` to `md->py`.
    - If `md->choice` is not -1, it sets the y-coordinate `*cy` to `md->py + 1 + md->choice`.
    - The function returns a pointer to the `screen` structure within the `menu_data` structure.
- **Output**:
    - A pointer to the `screen` structure associated with the menu data.


---
### menu_check_cb<!-- {{#callable:menu_check_cb}} -->
The `menu_check_cb` function calculates the unobstructed parts of an input range that are not covered by a menu overlay.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `data`: A pointer to a `menu_data` structure, which contains information about the menu.
    - `px`: The x-coordinate of the starting point of the input range.
    - `py`: The y-coordinate of the starting point of the input range.
    - `nx`: The width of the input range.
    - `r`: A pointer to an `overlay_ranges` structure where the unobstructed range will be stored.
- **Control Flow**:
    - Retrieve the `menu_data` structure from the `data` pointer.
    - Access the `menu` structure from the `menu_data`.
    - Call [`server_client_overlay_range`](server-client.c.driver.md#server_client_overlay_range) with the menu's position and dimensions, along with the input range coordinates and dimensions, to calculate the unobstructed range.
- **Output**:
    - The function does not return a value but modifies the `overlay_ranges` structure pointed to by `r` to reflect the parts of the input range not covered by the menu.
- **Functions called**:
    - [`server_client_overlay_range`](server-client.c.driver.md#server_client_overlay_range)


---
### menu_draw_cb<!-- {{#callable:menu_draw_cb}} -->
The `menu_draw_cb` function is responsible for rendering a menu on the screen for a given client, using the provided menu data and screen redraw context.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the menu is being drawn.
    - `data`: A pointer to a `menu_data` structure containing the menu information and styles to be used for drawing.
    - `rctx`: An unused pointer to a `screen_redraw_ctx` structure, which is not utilized in this function.
- **Control Flow**:
    - Initialize a `screen_write_ctx` structure for writing to the screen associated with the menu data.
    - Clear the screen using [`screen_write_clearscreen`](screen-write.c.driver.md#screen_write_clearscreen) with a specified parameter (8).
    - Check if border lines are specified in the menu data; if so, draw a box around the menu using [`screen_write_box`](screen-write.c.driver.md#screen_write_box).
    - Render the menu items on the screen using [`screen_write_menu`](screen-write.c.driver.md#screen_write_menu), applying the specified styles from the menu data.
    - Stop the screen writing context using [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop).
    - Iterate over each line of the screen and draw it on the client's terminal using [`tty_draw_line`](tty.c.driver.md#tty_draw_line), adjusting for the menu's position and dimensions.
- **Output**:
    - The function does not return a value; it performs its operations directly on the screen and terminal associated with the client.
- **Functions called**:
    - [`screen_write_start`](screen-write.c.driver.md#screen_write_start)
    - [`screen_write_clearscreen`](screen-write.c.driver.md#screen_write_clearscreen)
    - [`screen_write_box`](screen-write.c.driver.md#screen_write_box)
    - [`screen_write_menu`](screen-write.c.driver.md#screen_write_menu)
    - [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop)
    - [`tty_draw_line`](tty.c.driver.md#tty_draw_line)


---
### menu_free_cb<!-- {{#callable:menu_free_cb}} -->
The `menu_free_cb` function is responsible for cleaning up and freeing resources associated with a menu data structure.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `data`: A pointer to a `menu_data` structure that contains the menu and related data to be freed.
- **Control Flow**:
    - Cast the `data` pointer to a `menu_data` structure pointer `md`.
    - Check if `md->item` is not NULL, and if so, call [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue) with `md->item` to continue the command queue item.
    - Check if `md->cb` is not NULL, and if so, call the callback `md->cb` with parameters `md->menu`, `UINT_MAX`, `KEYC_NONE`, and `md->data`.
    - Call [`screen_free`](screen.c.driver.md#screen_free) to free the screen associated with `md->s`.
    - Call [`menu_free`](#menu_free) to free the menu associated with `md->menu`.
    - Free the `md` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided `menu_data` structure.
- **Functions called**:
    - [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue)
    - [`screen_free`](screen.c.driver.md#screen_free)
    - [`menu_free`](#menu_free)


---
### menu_key_cb<!-- {{#callable:menu_key_cb}} -->
The `menu_key_cb` function handles key and mouse events for navigating and selecting items in a menu within a client interface.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client interface where the menu is displayed.
    - `data`: A pointer to `menu_data` structure which contains information about the menu, such as the current choice, menu items, and callback functions.
    - `event`: A pointer to a `key_event` structure that contains information about the key or mouse event that triggered the callback.
- **Control Flow**:
    - Check if the event is a mouse event using `KEYC_IS_MOUSE` macro.
    - If mouse events are not allowed (`MENU_NOMOUSE` flag), only allow left mouse button clicks to proceed.
    - Check if the mouse click is within the menu boundaries; if not, handle based on `MENU_STAYOPEN` flag and update choice if necessary.
    - If the mouse click is within the menu, update the current choice based on the mouse position and redraw the overlay if the choice has changed.
    - For key events, iterate through menu items to find a matching key and update the choice if found.
    - Handle navigation keys (up, down, page up, page down, home, end) to change the current choice and redraw the overlay if necessary.
    - Handle special keys like backspace, tab, enter, escape, and control keys to perform specific actions or exit the menu.
    - If a valid choice is made, execute the associated command or callback function, and handle any command parsing errors.
- **Output**:
    - Returns 1 if the menu should be closed or an action is completed, otherwise returns 0 to keep the menu open.
- **Functions called**:
    - [`cmdq_get_event`](cmd-queue.c.driver.md#cmdq_get_event)
    - [`cmdq_new_state`](cmd-queue.c.driver.md#cmdq_new_state)
    - [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append)
    - [`cmdq_get_error`](cmd-queue.c.driver.md#cmdq_get_error)
    - [`cmdq_free_state`](cmd-queue.c.driver.md#cmdq_free_state)


---
### menu_set_style<!-- {{#callable:menu_set_style}} -->
The `menu_set_style` function configures the style of a menu item by applying default and specified styles to a grid cell.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the menu style is being set.
    - `gc`: A pointer to a `grid_cell` structure where the style will be applied.
    - `style`: A string representing the style to be applied, which can be NULL.
    - `option`: A string representing the option name used to apply the style from the client's options.
- **Control Flow**:
    - Copy the default grid cell style into the provided grid cell `gc`.
    - Apply the style from the client's options to the grid cell `gc` using the specified `option`.
    - If a `style` string is provided, initialize a temporary style structure `sytmp` with the default grid cell style.
    - Parse the provided `style` string into the temporary style structure `sytmp` and apply it to the grid cell `gc` if parsing is successful.
    - Update the foreground and background colors of `gc` with those from `sytmp` if the style parsing was successful.
- **Output**:
    - The function does not return a value; it modifies the `grid_cell` structure pointed to by `gc` to reflect the applied styles.
- **Functions called**:
    - [`style_apply`](style.c.driver.md#style_apply)
    - [`style_set`](style.c.driver.md#style_set)
    - [`style_parse`](style.c.driver.md#style_parse)


---
### menu_prepare<!-- {{#callable:menu_prepare}} -->
The `menu_prepare` function initializes and prepares a menu for display, setting its styles, position, and initial choice based on the provided parameters.
- **Inputs**:
    - `menu`: A pointer to a `struct menu` which contains the menu items and dimensions.
    - `flags`: An integer representing various flags that control menu behavior.
    - `starting_choice`: An integer indicating the initial choice index in the menu.
    - `item`: A pointer to a `struct cmdq_item` representing the command queue item associated with the menu.
    - `px`: An unsigned integer representing the x-coordinate for the menu's position.
    - `py`: An unsigned integer representing the y-coordinate for the menu's position.
    - `c`: A pointer to a `struct client` representing the client for which the menu is being prepared.
    - `lines`: An enum value of type `box_lines` indicating the style of the menu's border lines.
    - `style`: A string representing the style to be applied to the menu.
    - `selected_style`: A string representing the style to be applied to the selected menu item.
    - `border_style`: A string representing the style to be applied to the menu's border.
    - `fs`: A pointer to a `struct cmd_find_state` used to copy the command find state if provided.
    - `cb`: A callback function of type `menu_choice_cb` to be called when a menu choice is made.
    - `data`: A pointer to user-defined data to be passed to the callback function.
- **Control Flow**:
    - Check if the menu can fit within the client's terminal dimensions; if not, return NULL.
    - Adjust the menu's position if it overflows the terminal's dimensions.
    - Set the border lines style based on the provided `lines` parameter or default options.
    - Allocate and initialize a `menu_data` structure to hold the menu's state and styles.
    - Set the menu's styles using the [`menu_set_style`](#menu_set_style) function for the general, selected, and border styles.
    - Copy the command find state if `fs` is provided.
    - Initialize the screen for the menu with the calculated dimensions and set the mode based on the `MENU_NOMOUSE` flag.
    - Set the initial choice in the menu based on the `starting_choice` parameter, considering non-selectable items.
    - Assign the callback function and user data to the `menu_data` structure.
    - Return the initialized `menu_data` structure.
- **Output**:
    - A pointer to a `struct menu_data` containing the prepared menu's state, or NULL if the menu cannot fit within the terminal dimensions.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`menu_set_style`](#menu_set_style)
    - [`cmd_find_copy_state`](cmd-find.c.driver.md#cmd_find_copy_state)
    - [`screen_init`](screen.c.driver.md#screen_init)


---
### menu_display<!-- {{#callable:menu_display}} -->
The `menu_display` function prepares and displays a menu overlay on a client interface, allowing user interaction with menu items.
- **Inputs**:
    - `menu`: A pointer to a `struct menu` which contains the menu items and title to be displayed.
    - `flags`: An integer representing various flags that control menu behavior.
    - `starting_choice`: An integer indicating the initial menu item to be selected.
    - `item`: A pointer to a `struct cmdq_item` representing the command queue item associated with the menu.
    - `px`: An unsigned integer representing the x-coordinate for the menu's position.
    - `py`: An unsigned integer representing the y-coordinate for the menu's position.
    - `c`: A pointer to a `struct client` representing the client on which the menu is displayed.
    - `lines`: An enum `box_lines` indicating the style of the menu's border lines.
    - `style`: A string representing the style to be applied to the menu.
    - `selected_style`: A string representing the style to be applied to the selected menu item.
    - `border_style`: A string representing the style to be applied to the menu's border.
    - `fs`: A pointer to a `struct cmd_find_state` used for command state management.
    - `cb`: A callback function of type `menu_choice_cb` to be called when a menu choice is made.
    - `data`: A pointer to user-defined data to be passed to the callback function.
- **Control Flow**:
    - Call [`menu_prepare`](#menu_prepare) to initialize a `struct menu_data` with the provided parameters.
    - Check if [`menu_prepare`](#menu_prepare) returns NULL, indicating a failure to prepare the menu, and return -1 in this case.
    - If [`menu_prepare`](#menu_prepare) is successful, call [`server_client_set_overlay`](server-client.c.driver.md#server_client_set_overlay) to set the menu as an overlay on the client, using various callback functions for menu interaction.
    - Return 0 to indicate successful display of the menu.
- **Output**:
    - Returns 0 on successful display of the menu, or -1 if the menu could not be prepared.
- **Functions called**:
    - [`menu_prepare`](#menu_prepare)
    - [`server_client_set_overlay`](server-client.c.driver.md#server_client_set_overlay)


