# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides functionality for creating and managing interactive menus within the tmux environment. The code defines a set of functions and data structures that handle the creation, display, and interaction of menu items, allowing users to navigate and select options using keyboard or mouse inputs. The primary data structure, `menu_data`, encapsulates the state and properties of a menu, including its style, position, and the list of items it contains. The file includes functions for adding items to a menu, rendering the menu on the screen, handling user input to navigate and select menu items, and cleaning up resources when the menu is no longer needed.

The code is designed to be integrated into the tmux application, providing a modular and reusable component for menu management. It includes functions like [`menu_create`](#menu_create), [`menu_add_item`](#menu_add_item), and [`menu_display`](#menu_display), which are responsible for initializing menus, adding items, and displaying them to the user, respectively. The file also defines callback functions for handling user interactions, such as [`menu_key_cb`](#menu_key_cb) for keyboard events and [`menu_draw_cb`](#menu_draw_cb) for rendering the menu. The code is structured to support customization of menu appearance through styles and options, and it handles various input scenarios, including mouse interactions and keyboard shortcuts, to enhance user experience within the tmux environment.
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
    - `data`: A pointer to additional data passed to the callback function.
- **Description**: The `menu_data` structure is used to manage the state and appearance of a menu in a terminal-based application. It includes pointers to command queue items and menu structures, style information for the menu and its borders, and coordinates for positioning the menu on the screen. The structure also holds a callback function and associated data for handling user interactions with the menu.


# Functions

---
### menu_add_items<!-- {{#callable:menu_add_items}} -->
The `menu_add_items` function iterates over a list of menu items and adds each item to a specified menu using the [`menu_add_item`](#menu_add_item) function.
- **Inputs**:
    - `menu`: A pointer to the `menu` structure where items will be added.
    - `items`: A pointer to an array of `menu_item` structures representing the items to be added to the menu.
    - `qitem`: A pointer to a `cmdq_item` structure, which may be used for command queue operations.
    - `c`: A pointer to a `client` structure, representing the client context in which the menu is being used.
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
    - `item`: A pointer to the `menu_item` structure containing the name, command, and key of the item to be added.
    - `qitem`: A pointer to the `cmdq_item` structure used for formatting the item's name and command.
    - `c`: A pointer to the `client` structure representing the client context, used for determining available space and formatting.
    - `fs`: A pointer to the `cmd_find_state` structure used for formatting the item's name and command if provided.
- **Control Flow**:
    - Check if the item is a line separator or if the menu is empty, and return if true.
    - Reallocate memory to accommodate the new item in the menu's items array.
    - Initialize the new item structure with default values.
    - Format the item's name using `format_single_from_state` or `format_single` based on the presence of `fs`.
    - If the formatted name is empty, decrement the menu count and return.
    - Calculate the maximum width available for the item name based on the client's terminal size.
    - Determine if a key should be displayed with the item based on its length and available space.
    - Trim the formatted name to fit within the maximum width, appending a suffix if necessary.
    - Format the final display name of the item, including the key if applicable, and assign it to the new item.
    - Format the item's command using `format_single_from_state` or `format_single` based on the presence of `fs`, and assign it to the new item.
    - Update the menu's width if the new item's width exceeds the current menu width.
- **Output**:
    - The function does not return a value; it modifies the `menu` structure by adding a new formatted item to it.


---
### menu_create<!-- {{#callable:menu_create}} -->
The `menu_create` function initializes and returns a new menu structure with a specified title.
- **Inputs**:
    - `title`: A constant character pointer representing the title of the menu to be created.
- **Control Flow**:
    - Allocate memory for a new `menu` structure using `xcalloc` and assign it to the `menu` pointer.
    - Duplicate the input `title` string using `xstrdup` and assign it to the `title` field of the `menu` structure.
    - Calculate the width of the `title` using `format_width` and assign it to the `width` field of the `menu` structure.
    - Return the pointer to the newly created `menu` structure.
- **Output**:
    - A pointer to the newly created `menu` structure, initialized with the specified title and its width.


---
### menu_free<!-- {{#callable:menu_free}} -->
The `menu_free` function deallocates memory associated with a `menu` structure, including its items and title.
- **Inputs**:
    - `menu`: A pointer to a `struct menu` that needs to be freed.
- **Control Flow**:
    - Iterate over each item in the `menu` using a loop that runs from 0 to `menu->count - 1`.
    - For each item, free the memory allocated for `name` and `command` fields.
    - Free the memory allocated for the `items` array itself.
    - Free the memory allocated for the `title` of the menu.
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
    - Retrieve the `menu_data` structure from the `data` pointer.
    - Set the x-coordinate `*cx` to the menu's x-position `px` plus 2.
    - Check if the current choice `choice` is -1 (indicating no selection).
    - If `choice` is -1, set the y-coordinate `*cy` to the menu's y-position `py`.
    - If `choice` is not -1, set the y-coordinate `*cy` to `py` plus 1 plus the current choice index.
    - Return a pointer to the screen `s` associated with the menu data.
- **Output**:
    - A pointer to the `screen` structure associated with the menu data, which represents the screen where the menu is displayed.


---
### menu_check_cb<!-- {{#callable:menu_check_cb}} -->
The `menu_check_cb` function calculates and returns the unobstructed parts of an input range that are not covered by a menu overlay.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `data`: A pointer to a `menu_data` structure, which contains information about the menu.
    - `px`: The x-coordinate of the starting point of the input range.
    - `py`: The y-coordinate of the starting point of the input range.
    - `nx`: The width of the input range.
    - `r`: A pointer to an `overlay_ranges` structure where the unobstructed parts of the range will be stored.
- **Control Flow**:
    - Retrieve the `menu_data` structure from the `data` pointer.
    - Access the `menu` structure from the `menu_data`.
    - Call `server_client_overlay_range` with the menu's position and dimensions, along with the input range coordinates and dimensions, to calculate the unobstructed parts of the range.
- **Output**:
    - The function does not return a value but modifies the `overlay_ranges` structure pointed to by `r` to store the unobstructed parts of the input range.


---
### menu_draw_cb<!-- {{#callable:menu_draw_cb}} -->
The `menu_draw_cb` function is responsible for rendering a menu on the screen for a given client.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the menu is being drawn.
    - `data`: A pointer to a `menu_data` structure containing the menu information and rendering context.
    - `rctx`: An unused pointer to a `screen_redraw_ctx` structure, which is not utilized in this function.
- **Control Flow**:
    - Initialize a `screen_write_ctx` structure for writing to the screen associated with the menu data.
    - Clear the screen using `screen_write_clearscreen` with a specific flag (8).
    - Check if border lines are specified in the menu data; if so, draw a box around the menu using `screen_write_box`.
    - Render the menu items on the screen using `screen_write_menu`, applying styles for normal, border, and selected items.
    - Stop the screen writing context using `screen_write_stop`.
    - Iterate over each line of the screen and draw it on the client's terminal using `tty_draw_line`, adjusting for the menu's position and dimensions.
- **Output**:
    - The function does not return a value; it performs its operations directly on the screen and terminal associated with the client.


---
### menu_free_cb<!-- {{#callable:menu_free_cb}} -->
The `menu_free_cb` function is responsible for cleaning up and freeing resources associated with a menu data structure.
- **Inputs**:
    - `c`: A pointer to a `struct client`, which is unused in this function.
    - `data`: A pointer to a `struct menu_data`, which contains the menu data to be freed.
- **Control Flow**:
    - Cast the `data` pointer to a `struct menu_data` pointer named `md`.
    - Check if `md->item` is not NULL; if so, call `cmdq_continue` with `md->item`.
    - Check if `md->cb` is not NULL; if so, call `md->cb` with `md->menu`, `UINT_MAX`, `KEYC_NONE`, and `md->data`.
    - Call `screen_free` to free the screen associated with `md->s`.
    - Call [`menu_free`](#menu_free) to free the menu associated with `md->menu`.
    - Free the memory allocated for `md`.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided menu data.
- **Functions called**:
    - [`menu_free`](#menu_free)


---
### menu_key_cb<!-- {{#callable:menu_key_cb}} -->
The `menu_key_cb` function handles key and mouse events for a menu, updating the menu's state and executing commands based on user interactions.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client interacting with the menu.
    - `data`: A pointer to `menu_data` structure containing the menu's state and configuration.
    - `event`: A pointer to a `key_event` structure representing the key or mouse event to be processed.
- **Control Flow**:
    - Check if the event is a mouse event using `KEYC_IS_MOUSE` and handle mouse interactions with the menu, including checking if the mouse is within the menu bounds and updating the menu choice accordingly.
    - If the event is not a mouse event, iterate over the menu items to find a matching key event and update the menu choice if a match is found.
    - Handle specific key events such as arrow keys, page up/down, home/end, and others to navigate the menu and update the choice.
    - If a valid menu item is chosen, execute the associated command using `cmd_parse_and_append` and handle any errors.
    - Return 1 to indicate the menu should close or 0 to keep it open based on the event and menu state.
- **Output**:
    - Returns an integer indicating whether the menu should remain open (0) or close (1) based on the processed event.


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


---
### menu_prepare<!-- {{#callable:menu_prepare}} -->
The `menu_prepare` function initializes and configures a menu data structure for display, setting styles, positions, and handling initial choice selection.
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
    - Allocate and initialize a `menu_data` structure, setting its fields with the provided parameters.
    - Apply styles to the menu, selected item, and border using the [`menu_set_style`](#menu_set_style) function.
    - Copy the command find state if `fs` is not NULL.
    - Initialize a screen for the menu with dimensions based on the menu's width and item count.
    - Set the screen mode to enable mouse interaction unless the `MENU_NOMOUSE` flag is set.
    - Determine the initial choice based on the `starting_choice` parameter and the `MENU_NOMOUSE` flag.
    - Assign the callback function and user data to the `menu_data` structure.
    - Return the initialized `menu_data` structure.
- **Output**:
    - Returns a pointer to a `struct menu_data` containing the prepared menu data, or NULL if the menu cannot fit within the client's terminal dimensions.
- **Functions called**:
    - [`menu_set_style`](#menu_set_style)


---
### menu_display<!-- {{#callable:menu_display}} -->
The `menu_display` function initializes and displays a menu overlay on a client interface, handling user interactions and menu item selections.
- **Inputs**:
    - `menu`: A pointer to a `struct menu` representing the menu to be displayed.
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
    - `cb`: A callback function of type `menu_choice_cb` to be called upon menu item selection.
    - `data`: A pointer to user-defined data to be passed to the callback function.
- **Control Flow**:
    - Call [`menu_prepare`](#menu_prepare) to initialize a `struct menu_data` with the provided parameters.
    - Check if [`menu_prepare`](#menu_prepare) returns NULL, indicating a failure to prepare the menu, and return -1 in this case.
    - If [`menu_prepare`](#menu_prepare) is successful, call `server_client_set_overlay` to set up the menu overlay on the client, passing various callback functions for menu interaction handling.
    - Return 0 to indicate successful menu display setup.
- **Output**:
    - Returns 0 on successful menu display setup, or -1 if the menu could not be prepared.
- **Functions called**:
    - [`menu_prepare`](#menu_prepare)


