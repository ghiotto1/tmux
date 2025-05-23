# Purpose
This C source code file is part of the `tmux` project, specifically implementing a window mode for customizing options and key bindings within the `tmux` terminal multiplexer. The file defines a mode called "options-mode," which allows users to interactively view and modify various configuration options and key bindings associated with the `tmux` server, sessions, windows, and panes. The code provides a structured interface for displaying these options in a tree-like format, enabling users to navigate, select, and edit options or key bindings directly from the terminal.

Key components of the code include the definition of the `window_customize_mode` structure, which specifies the initialization, resizing, key handling, and cleanup functions for this mode. The code also defines several enumerations and data structures to manage the scope and state of customizable items, such as `window_customize_scope` and `window_customize_itemdata`. The file includes functions for building the options and key bindings tree, handling user input to modify settings, and rendering the interface. This code is intended to be integrated into the `tmux` application, providing a user interface for customizing its behavior without directly editing configuration files.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### window_customize_init
- **Type**: ``struct screen *``
- **Description**: The `window_customize_init` function is a static function that initializes a `struct screen` for a window mode entry. It sets up the necessary data structures and configurations for customizing window options in a terminal multiplexer environment.
- **Use**: This function is used to initialize the screen and related data structures for a window mode entry, allowing for customization of window options.


---
### window_customize_menu_items
- **Type**: ``const struct menu_item[]``
- **Description**: `window_customize_menu_items` is a static constant array of `menu_item` structures. Each element in the array represents a menu item with a label, a key code, and a NULL pointer for additional data.
- **Use**: This array is used to define the menu items available in the window customization menu, providing labels and key codes for user interaction.


---
### window_customize_mode
- **Type**: `const struct window_mode`
- **Description**: The `window_customize_mode` is a constant structure of type `window_mode` that defines a specific mode for customizing window options in the application. It includes a name, a default format for displaying options, and function pointers for initializing, freeing, resizing, and handling key events in this mode.
- **Use**: This variable is used to define and manage the behavior of the window customization mode, allowing users to interact with and modify window options.


# Data Structures

---
### window_customize_scope
- **Type**: `enum`
- **Members**:
    - `WINDOW_CUSTOMIZE_NONE`: Represents no customization scope.
    - `WINDOW_CUSTOMIZE_KEY`: Represents customization scope for keys.
    - `WINDOW_CUSTOMIZE_SERVER`: Represents customization scope for the server.
    - `WINDOW_CUSTOMIZE_GLOBAL_SESSION`: Represents customization scope for global session settings.
    - `WINDOW_CUSTOMIZE_SESSION`: Represents customization scope for session settings.
    - `WINDOW_CUSTOMIZE_GLOBAL_WINDOW`: Represents customization scope for global window settings.
    - `WINDOW_CUSTOMIZE_WINDOW`: Represents customization scope for window settings.
    - `WINDOW_CUSTOMIZE_PANE`: Represents customization scope for pane settings.
- **Description**: The `window_customize_scope` enumeration defines various levels of customization scopes within a window management system, such as server, session, window, and pane levels. Each enumerator represents a specific scope where customization can be applied, allowing for fine-grained control over different aspects of the window management environment.


---
### window_customize_change
- **Type**: `enum`
- **Members**:
    - `WINDOW_CUSTOMIZE_UNSET`: Represents an unset state for window customization.
    - `WINDOW_CUSTOMIZE_RESET`: Represents a reset state for window customization.
- **Description**: The `window_customize_change` is an enumeration that defines the possible states for customizing a window in the context of the software. It includes two states: `WINDOW_CUSTOMIZE_UNSET`, which indicates that no customization has been set, and `WINDOW_CUSTOMIZE_RESET`, which indicates that the customization should be reset to its default state. This enumeration is likely used to manage and track the customization state of windows within the application.


---
### window_customize_itemdata
- **Type**: `struct`
- **Members**:
    - `data`: A pointer to a `window_customize_modedata` structure, representing the mode data associated with the item.
    - `scope`: An enumeration of type `window_customize_scope` that defines the scope of the customization.
    - `table`: A pointer to a character string representing the name of the key table associated with the item.
    - `key`: A `key_code` representing the key associated with the item.
    - `oo`: A pointer to an `options` structure, representing the options associated with the item.
    - `name`: A pointer to a character string representing the name of the option or key.
    - `idx`: An integer representing the index of the item, used for array options.
- **Description**: The `window_customize_itemdata` structure is used to represent an individual item in the window customization mode of a terminal multiplexer application. It holds information about the customization item, including its associated mode data, scope, key table, key code, options, name, and index. This structure is essential for managing and interacting with customizable options and key bindings within the application, allowing users to modify settings and key bindings dynamically.


---
### window_customize_modedata
- **Type**: `struct`
- **Members**:
    - `wp`: A pointer to a `window_pane` structure, representing the pane associated with this mode data.
    - `dead`: An integer flag indicating if the mode data is no longer active or valid.
    - `references`: An integer count of references to this mode data, used for memory management.
    - `data`: A pointer to a `mode_tree_data` structure, which holds the data for the mode tree.
    - `format`: A string representing the format used for displaying options or keys.
    - `hide_global`: An integer flag indicating whether global options should be hidden.
    - `prompt_flags`: An integer representing flags for prompt behavior.
    - `item_list`: A pointer to an array of pointers to `window_customize_itemdata` structures, representing the items in the mode.
    - `item_size`: An unsigned integer representing the number of items in the `item_list`.
    - `fs`: A `cmd_find_state` structure representing the state of a command find operation.
    - `change`: An enumeration value of type `window_customize_change`, indicating the type of change operation to perform.
- **Description**: The `window_customize_modedata` structure is used to manage the state and data associated with a customizable window mode in a terminal multiplexer. It holds references to the pane, mode tree data, and a list of items that can be customized, such as options or key bindings. The structure also includes flags and format strings to control the display and behavior of the customization interface, as well as a reference count for managing its lifecycle.


# Functions

---
### window_customize_get_tag<!-- {{#callable:window_customize_get_tag}} -->
The `window_customize_get_tag` function generates a unique 64-bit tag for an options entry, based on its index and position in the options table.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure, representing the options entry for which the tag is being generated.
    - `idx`: An integer representing the index of the options entry, used in the tag generation.
    - `oe`: A pointer to an `options_table_entry` structure, representing the entry in the options table; it can be NULL.
- **Control Flow**:
    - Check if `oe` is NULL; if so, return the casted pointer of `o` as a 64-bit unsigned integer.
    - Calculate the offset of `oe` from the start of the `options_table` by subtracting the base address of `options_table` from `oe` and dividing by the size of an entry in `options_table`.
    - Construct the tag by combining several bit-shifted components: a constant value `2ULL << 62`, the calculated offset shifted left by 32 bits, the index incremented by 1 and shifted left by 1 bit, and finally OR with 1.
    - Return the constructed 64-bit tag.
- **Output**:
    - A 64-bit unsigned integer representing a unique tag for the given options entry.


---
### window_customize_get_tree<!-- {{#callable:window_customize_get_tree}} -->
The function `window_customize_get_tree` returns a pointer to a specific options structure based on the given customization scope and command find state.
- **Inputs**:
    - `scope`: An enumeration value of type `enum window_customize_scope` that specifies the scope of customization, such as server, session, window, or pane.
    - `fs`: A pointer to a `struct cmd_find_state` which contains the current state of command finding, including session, window, and pane information.
- **Control Flow**:
    - The function uses a switch statement to determine the action based on the `scope` parameter.
    - If the `scope` is `WINDOW_CUSTOMIZE_NONE` or `WINDOW_CUSTOMIZE_KEY`, the function returns `NULL`.
    - If the `scope` is `WINDOW_CUSTOMIZE_SERVER`, it returns `global_options`.
    - If the `scope` is `WINDOW_CUSTOMIZE_GLOBAL_SESSION`, it returns `global_s_options`.
    - If the `scope` is `WINDOW_CUSTOMIZE_SESSION`, it returns the options from the session in `fs`.
    - If the `scope` is `WINDOW_CUSTOMIZE_GLOBAL_WINDOW`, it returns `global_w_options`.
    - If the `scope` is `WINDOW_CUSTOMIZE_WINDOW`, it returns the options from the window in `fs`.
    - If the `scope` is `WINDOW_CUSTOMIZE_PANE`, it returns the options from the pane in `fs`.
    - If none of the cases match, it returns `NULL`.
- **Output**:
    - A pointer to a `struct options` corresponding to the specified scope, or `NULL` if the scope is not applicable.


---
### window_customize_check_item<!-- {{#callable:window_customize_check_item}} -->
The `window_customize_check_item` function checks if a given item in a window customization mode matches the expected options tree based on the current find state.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure containing the current mode data, including the find state and window pane information.
    - `item`: A pointer to a `window_customize_itemdata` structure representing the item to be checked, including its options and scope.
    - `fsp`: A pointer to a `cmd_find_state` structure representing the find state; if NULL, a local find state is used.
- **Control Flow**:
    - A local `cmd_find_state` structure `fs` is declared.
    - If `fsp` is NULL, it is set to point to the local `fs`.
    - The function checks if the find state in `data` is valid using `cmd_find_valid_state`.
    - If valid, it copies the find state from `data` to `fsp` using `cmd_find_copy_state`.
    - If not valid, it initializes `fsp` from the pane in `data` using `cmd_find_from_pane`.
    - The function returns whether the options owner (`oo`) of `item` matches the options tree obtained from [`window_customize_get_tree`](#window_customize_get_tree) with `item->scope` and `fsp`.
- **Output**:
    - Returns an integer indicating whether the item's options owner matches the expected options tree for the given scope and find state.
- **Functions called**:
    - [`window_customize_get_tree`](#window_customize_get_tree)


---
### window_customize_get_key<!-- {{#callable:window_customize_get_key}} -->
The `window_customize_get_key` function retrieves a key binding from a specified key table and optionally returns pointers to the key table and key binding if they are found.
- **Inputs**:
    - `item`: A pointer to a `window_customize_itemdata` structure that contains the key table name and key code to search for.
    - `ktp`: A pointer to a pointer to a `key_table` structure where the found key table will be stored, if not NULL.
    - `bdp`: A pointer to a pointer to a `key_binding` structure where the found key binding will be stored, if not NULL.
- **Control Flow**:
    - Retrieve the key table using `key_bindings_get_table` with the table name from `item` and a flag set to 0.
    - If the key table is NULL, return 0 indicating failure to find the key table.
    - Retrieve the key binding using `key_bindings_get` with the retrieved key table and the key from `item`.
    - If the key binding is NULL, return 0 indicating failure to find the key binding.
    - If `ktp` is not NULL, store the retrieved key table in the location pointed to by `ktp`.
    - If `bdp` is not NULL, store the retrieved key binding in the location pointed to by `bdp`.
    - Return 1 indicating success in finding both the key table and key binding.
- **Output**:
    - Returns 1 if both the key table and key binding are found, otherwise returns 0.


---
### window_customize_scope_text<!-- {{#callable:window_customize_scope_text}} -->
The `window_customize_scope_text` function generates a descriptive string based on the given customization scope and command find state.
- **Inputs**:
    - `scope`: An enumeration value of type `enum window_customize_scope` that specifies the scope of customization (e.g., pane, session, window).
    - `fs`: A pointer to a `struct cmd_find_state` which contains information about the current command state, including session, window, and pane details.
- **Control Flow**:
    - The function begins by declaring a character pointer `s` and an unsigned integer `idx`.
    - A switch statement is used to determine the behavior based on the `scope` argument.
    - If the scope is `WINDOW_CUSTOMIZE_PANE`, it retrieves the pane index using `window_pane_index` and formats the string as 'pane <index>'.
    - If the scope is `WINDOW_CUSTOMIZE_SESSION`, it formats the string as 'session <session_name>'.
    - If the scope is `WINDOW_CUSTOMIZE_WINDOW`, it formats the string as 'window <window_index>'.
    - For any other scope, it defaults to an empty string.
    - The function returns the formatted string `s`.
- **Output**:
    - A dynamically allocated string that describes the scope, such as 'pane <index>', 'session <name>', or 'window <index>', or an empty string if the scope is not recognized.


---
### window_customize_add_item<!-- {{#callable:window_customize_add_item}} -->
The `window_customize_add_item` function adds a new item to the `item_list` of a `window_customize_modedata` structure, reallocating memory as necessary.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure, which contains the list of items and its current size.
- **Control Flow**:
    - Reallocate memory for `data->item_list` to accommodate one more `window_customize_itemdata` structure.
    - Increment `data->item_size` to reflect the addition of a new item.
    - Allocate memory for a new `window_customize_itemdata` structure and assign it to the newly added position in `data->item_list`.
    - Return the pointer to the newly created `window_customize_itemdata` structure.
- **Output**:
    - A pointer to the newly created `window_customize_itemdata` structure.


---
### window_customize_free_item<!-- {{#callable:window_customize_free_item}} -->
The `window_customize_free_item` function deallocates memory associated with a `window_customize_itemdata` structure.
- **Inputs**:
    - `item`: A pointer to a `window_customize_itemdata` structure that needs to be freed.
- **Control Flow**:
    - The function calls `free` on the `table` member of the `item` structure to release its allocated memory.
    - It then calls `free` on the `name` member of the `item` structure to release its allocated memory.
    - Finally, it calls `free` on the `item` itself to release the memory allocated for the structure.
- **Output**:
    - The function does not return any value; it performs memory deallocation for the given structure and its members.


---
### window_customize_build_array<!-- {{#callable:window_customize_build_array}} -->
The `window_customize_build_array` function iterates over an options array, formats each option's name and value, and adds them to a mode tree for display in a customizable window.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure that holds the mode data for the window customization.
    - `top`: A pointer to a `mode_tree_item` structure representing the top item in the mode tree where new items will be added.
    - `scope`: An enum value of type `window_customize_scope` indicating the scope of the customization (e.g., server, session, window, etc.).
    - `o`: A pointer to an `options_entry` structure representing the options entry to be processed.
    - `ft`: A pointer to a `format_tree` structure used for formatting the option's name and value.
- **Control Flow**:
    - Retrieve the options table entry and owner for the given options entry `o`.
    - Initialize the first item in the options array using `options_array_first`.
    - Iterate over each item in the options array using a while loop.
    - For each item, retrieve its index and format its name and value using `xasprintf` and `options_to_string`.
    - Add the formatted name and value to the format tree `ft`.
    - Create a new `window_customize_itemdata` item and set its properties, including scope, owner, name, and index.
    - Expand the format tree to generate a text representation of the item.
    - Calculate a unique tag for the item using [`window_customize_get_tag`](#window_customize_get_tag).
    - Add the item to the mode tree using `mode_tree_add`.
    - Free the allocated memory for the text, name, and value strings.
    - Move to the next item in the options array using `options_array_next`.
- **Output**:
    - The function does not return a value; it modifies the mode tree by adding formatted options array items to it.
- **Functions called**:
    - [`options_table_entry`](tmux.h.driver.md#options_table_entry)
    - [`window_customize_add_item`](#window_customize_add_item)
    - [`window_customize_get_tag`](#window_customize_get_tag)


---
### window_customize_build_option<!-- {{#callable:window_customize_build_option}} -->
The `window_customize_build_option` function constructs and adds an option item to a mode tree, considering various attributes and filters, for a given customization scope in a window customization mode.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure that holds the customization mode data.
    - `top`: A pointer to a `mode_tree_item` structure representing the top item in the mode tree where the option will be added.
    - `scope`: An enum value of type `window_customize_scope` indicating the scope of the option (e.g., server, session, window, etc.).
    - `o`: A pointer to an `options_entry` structure representing the option to be customized.
    - `ft`: A pointer to a `format_tree` structure used for formatting option attributes.
    - `filter`: A constant character pointer to a filter string used to determine if the option should be included.
    - `fs`: A pointer to a `cmd_find_state` structure used to find the current command state.
- **Control Flow**:
    - Retrieve the options table entry and owner for the given option `o`.
    - Check if the option is a hook or an array and set flags accordingly.
    - Determine if the option is global based on the scope and the `hide_global` flag in `data`.
    - Add formatted option attributes (name, global status, array status, scope, unit) to the format tree `ft`.
    - If the option is not an array, convert its value to a string and add it to the format tree.
    - If a filter is provided, expand it using the format tree and check if it evaluates to true; if not, return early.
    - Add a new item to the mode tree using [`window_customize_add_item`](#window_customize_add_item), setting its attributes and adding it to the mode tree with `mode_tree_add`.
    - If the option is an array, call [`window_customize_build_array`](#window_customize_build_array) to handle array elements.
- **Output**:
    - The function does not return a value; it modifies the mode tree by adding a new option item based on the provided parameters.
- **Functions called**:
    - [`options_table_entry`](tmux.h.driver.md#options_table_entry)
    - [`window_customize_scope_text`](#window_customize_scope_text)
    - [`window_customize_add_item`](#window_customize_add_item)
    - [`window_customize_get_tag`](#window_customize_get_tag)
    - [`window_customize_build_array`](#window_customize_build_array)


---
### window_customize_find_user_options<!-- {{#callable:window_customize_find_user_options}} -->
The function `window_customize_find_user_options` iterates through a list of options and appends user-defined options (those starting with '@') to a provided list if they are not already present.
- **Inputs**:
    - `oo`: A pointer to a struct `options` which represents the collection of options to be searched.
    - `list`: A pointer to a pointer to a list of strings, which will be updated to include user-defined options found.
    - `size`: A pointer to an unsigned integer representing the current size of the list, which will be updated as new options are added.
- **Control Flow**:
    - Initialize an options entry pointer `o` to the first option in the `oo` options list.
    - Iterate over each option `o` in the options list `oo`.
    - For each option, retrieve its name and check if it starts with '@'.
    - If the name does not start with '@', continue to the next option.
    - If the name starts with '@', check if it is already in the `list` by iterating over the current list entries.
    - If the name is not found in the list, reallocate the list to accommodate one more entry, add the name to the list, and increment the size.
    - Continue to the next option until all options have been processed.
- **Output**:
    - The function updates the `list` to include user-defined options (those starting with '@') that are not already present, and updates the `size` to reflect the new number of entries in the list.


---
### window_customize_build_options<!-- {{#callable:window_customize_build_options}} -->
The `window_customize_build_options` function constructs a mode tree of customizable options from multiple option trees, applying filters and formatting as specified.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure that holds the mode data for customization.
    - `title`: A string representing the title of the mode tree.
    - `tag`: A unique identifier for the mode tree item.
    - `scope0`: The scope of the first options tree, specified as an enum `window_customize_scope`.
    - `oo0`: A pointer to the first options tree from which options are retrieved.
    - `scope1`: The scope of the second options tree, specified as an enum `window_customize_scope`.
    - `oo1`: A pointer to the second options tree, which may be NULL.
    - `scope2`: The scope of the third options tree, specified as an enum `window_customize_scope`.
    - `oo2`: A pointer to the third options tree, which may be NULL.
    - `ft`: A pointer to a `format_tree` structure used for formatting options.
    - `filter`: A string representing a filter to apply to the options, which may be NULL.
    - `fs`: A pointer to a `cmd_find_state` structure used to find the command state.
- **Control Flow**:
    - Initialize a mode tree item `top` with the given title and tag.
    - Find user options from the first options tree `oo0` and append them to a list.
    - If `oo1` and `oo2` are not NULL, find user options from these trees and append them to the list.
    - Iterate over the list of user options, retrieving each option from `oo2`, `oo1`, or `oo0` in that order.
    - Determine the scope of each option based on its owner and call [`window_customize_build_option`](#window_customize_build_option) to build the option in the mode tree.
    - Free the list of user options.
    - Iterate over all options in `oo0`, skipping those with names starting with '@'.
    - For each option, retrieve it from `oo2`, `oo1`, or `oo0` in that order, determine its scope, and call [`window_customize_build_option`](#window_customize_build_option) to build the option in the mode tree.
- **Output**:
    - The function does not return a value; it modifies the mode tree structure within the `window_customize_modedata`.
- **Functions called**:
    - [`window_customize_find_user_options`](#window_customize_find_user_options)
    - [`window_customize_build_option`](#window_customize_build_option)


---
### window_customize_build_keys<!-- {{#callable:window_customize_build_keys}} -->
The `window_customize_build_keys` function constructs a hierarchical mode tree of key bindings from a given key table, applying optional filters and formatting.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure that holds the mode data and state.
    - `kt`: A pointer to a `key_table` structure representing the key table to be processed.
    - `ft`: A pointer to a `format_tree` structure used for formatting the output.
    - `filter`: A string representing a filter to apply to the key bindings.
    - `fs`: A pointer to a `cmd_find_state` structure used for command state information.
    - `number`: An unsigned integer used to generate a unique tag for the mode tree items.
- **Control Flow**:
    - Generate a unique tag using the provided number and bitwise operations.
    - Create a title string for the key table and add a top-level item to the mode tree with this title.
    - Initialize a format tree from the command state and add key-related format variables.
    - Iterate over each key binding in the key table.
    - For each key binding, add its key and note to the format tree.
    - If a filter is provided, expand it and check if the key binding should be included; skip if not.
    - Create a new `window_customize_itemdata` item for the key binding and add it to the mode tree as a child of the top-level item.
    - Expand the format string for the key binding and add it to the mode tree.
    - Add command, note, and repeat status as children of the key binding item in the mode tree.
    - Free allocated memory for expanded strings and format trees.
- **Output**:
    - The function does not return a value; it modifies the mode tree structure within the `window_customize_modedata` object to include the key bindings from the specified key table.
- **Functions called**:
    - [`window_customize_add_item`](#window_customize_add_item)


---
### window_customize_build<!-- {{#callable:window_customize_build}} -->
The `window_customize_build` function initializes and populates a data structure with options and key bindings for a window customization mode, applying filters and formatting as specified.
- **Inputs**:
    - `modedata`: A pointer to a `window_customize_modedata` structure that holds the state and data for the window customization mode.
    - `sort_crit`: An unused parameter of type `struct mode_tree_sort_criteria`.
    - `tag`: An unused parameter of type `uint64_t`.
    - `filter`: A string used to filter the options and key bindings to be included in the customization mode.
- **Control Flow**:
    - The function starts by freeing any existing items in the `item_list` of the `modedata` structure and resets the list size to zero.
    - It checks if the `fs` state in `modedata` is valid; if so, it copies it to a local `fs` variable, otherwise, it initializes `fs` from the pane in `modedata`.
    - A format tree `ft` is created from the `fs` state, and initial format variables are set for options and keys.
    - The function calls [`window_customize_build_options`](#window_customize_build_options) three times to build options for server, session, and window & pane, using different scopes and option trees.
    - The format tree `ft` is freed and recreated from the `fs` state.
    - A loop iterates over key tables, calling [`window_customize_build_keys`](#window_customize_build_keys) for each non-empty key table, up to a maximum of 256 tables.
    - The format tree `ft` is freed at the end of the function.
- **Output**:
    - The function does not return a value; it modifies the `modedata` structure to populate it with formatted options and key bindings for the window customization mode.
- **Functions called**:
    - [`window_customize_free_item`](#window_customize_free_item)
    - [`window_customize_build_options`](#window_customize_build_options)
    - [`window_customize_build_keys`](#window_customize_build_keys)


---
### window_customize_draw_key<!-- {{#callable:window_customize_draw_key}} -->
The `window_customize_draw_key` function displays information about a specific key binding in a window customization context.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure, which is unused in this function.
    - `item`: A pointer to a `window_customize_itemdata` structure representing the key binding item to be drawn.
    - `ctx`: A pointer to a `screen_write_ctx` structure used for writing to the screen.
    - `sx`: An unsigned integer representing the width of the screen area available for drawing.
    - `sy`: An unsigned integer representing the height of the screen area available for drawing.
- **Control Flow**:
    - Check if the `item` is NULL or if the key binding cannot be retrieved using [`window_customize_get_key`](#window_customize_get_key); if so, return immediately.
    - Retrieve the note associated with the key binding and append a period if it doesn't already end with one.
    - Write the note to the screen using `screen_write_text`; if writing fails, return.
    - Move the cursor down one line using `screen_write_cursormove`; if the cursor is out of bounds, return.
    - Write the key table name and repeat status to the screen; if writing fails, return.
    - Move the cursor down one line again; if the cursor is out of bounds, return.
    - Retrieve and write the command associated with the key binding to the screen; if writing fails, free the command string and return.
    - Retrieve the default key binding and compare its command with the current command; if they differ, write the default command to the screen.
    - Free any allocated command strings before returning.
- **Output**:
    - The function does not return a value; it writes information about a key binding to the screen.
- **Functions called**:
    - [`window_customize_get_key`](#window_customize_get_key)


---
### window_customize_draw_option<!-- {{#callable:window_customize_draw_option}} -->
The `window_customize_draw_option` function displays detailed information about a specific option in a customizable window interface, including its description, scope, value, and other attributes.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure containing mode-specific data.
    - `item`: A pointer to a `window_customize_itemdata` structure representing the specific option item to be drawn.
    - `ctx`: A pointer to a `screen_write_ctx` structure used for writing to the screen.
    - `sx`: An unsigned integer representing the width of the screen area available for drawing.
    - `sy`: An unsigned integer representing the height of the screen area available for drawing.
- **Control Flow**:
    - Check if the item is valid using [`window_customize_check_item`](#window_customize_check_item); if not, return immediately.
    - Retrieve the option entry using `options_get` and check if it exists; if not, return.
    - Determine the unit and description of the option from the options table entry, if available.
    - Write the option's description to the screen using `screen_write_text`.
    - Determine the scope of the option (e.g., user, window, session, server) and write it to the screen.
    - If the option is an array, write the array index information to the screen.
    - Retrieve the current value of the option and write it to the screen, including any unit information.
    - If the option is a string, expand it using `format_expand` and write the expanded value if it differs from the original.
    - If the option is a choice, list all available choices and write them to the screen.
    - If the option is a color, apply the color to a grid cell and write an example text with the color applied.
    - If the option is a style, apply the style to a grid cell and write an example text with the style applied.
    - If a default value exists and differs from the current value, write the default value to the screen.
    - Check for parent options in window and global scopes and write their values if they differ from the current option.
    - Free any allocated memory for values and the format tree before exiting.
- **Output**:
    - The function does not return a value; it writes information about the option to the screen using the provided context.
- **Functions called**:
    - [`window_customize_check_item`](#window_customize_check_item)
    - [`options_table_entry`](tmux.h.driver.md#options_table_entry)


---
### window_customize_draw<!-- {{#callable:window_customize_draw}} -->
The `window_customize_draw` function determines whether to draw a key or an option in a customizable window based on the item's scope.
- **Inputs**:
    - `modedata`: A pointer to `window_customize_modedata` structure containing mode-specific data.
    - `itemdata`: A pointer to `window_customize_itemdata` structure representing the item to be drawn.
    - `ctx`: A pointer to `screen_write_ctx` structure used for writing to the screen.
    - `sx`: An unsigned integer representing the width of the screen area to draw in.
    - `sy`: An unsigned integer representing the height of the screen area to draw in.
- **Control Flow**:
    - Check if `item` is NULL and return immediately if true.
    - Determine the scope of the `item`.
    - If the scope is `WINDOW_CUSTOMIZE_KEY`, call [`window_customize_draw_key`](#window_customize_draw_key) to draw the key.
    - Otherwise, call [`window_customize_draw_option`](#window_customize_draw_option) to draw the option.
- **Output**:
    - The function does not return a value; it performs drawing operations on the screen context provided.
- **Functions called**:
    - [`window_customize_draw_key`](#window_customize_draw_key)
    - [`window_customize_draw_option`](#window_customize_draw_option)


---
### window_customize_menu<!-- {{#callable:window_customize_menu}} -->
The `window_customize_menu` function handles key events for customizing a window's menu in a terminal multiplexer.
- **Inputs**:
    - `modedata`: A pointer to the `window_customize_modedata` structure, which contains data related to the window customization mode.
    - `c`: A pointer to the `client` structure, representing the client interacting with the window.
    - `key`: A `key_code` representing the key event that triggered the function.
- **Control Flow**:
    - Retrieve the `window_customize_modedata` structure from the `modedata` pointer.
    - Access the `window_pane` from the `window_customize_modedata` structure.
    - Get the first `window_mode_entry` from the `window_pane`'s modes queue.
    - Check if the `window_mode_entry` is NULL or if its data does not match `modedata`; if so, return immediately.
    - Call the [`window_customize_key`](#window_customize_key) function with the `window_mode_entry`, client, and key to handle the key event.
- **Output**:
    - The function does not return any value; it performs actions based on the key event and modifies the state of the window customization mode.
- **Functions called**:
    - [`window_customize_key`](#window_customize_key)


---
### window_customize_height<!-- {{#callable:window_customize_height}} -->
The `window_customize_height` function returns a fixed height value of 12.
- **Inputs**:
    - `modedata`: This is a pointer to mode data, which is unused in this function.
    - `height`: This is an unsigned integer representing the height, which is unused in this function.
- **Control Flow**:
    - The function takes two parameters, both marked as unused, indicating they are not utilized within the function body.
    - The function directly returns the integer value 12 without any computation or condition checks.
- **Output**:
    - The function returns a constant unsigned integer value of 12.


---
### window_customize_init<!-- {{#callable:window_customize_init}} -->
The `window_customize_init` function initializes a customization mode for a window pane, setting up a mode tree for managing options and key bindings.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the mode entry for the window pane.
    - `fs`: A pointer to a `cmd_find_state` structure used to find the current state of the command.
    - `args`: A pointer to an `args` structure containing command-line arguments, which may include format and prompt flags.
- **Control Flow**:
    - Allocate memory for a `window_customize_modedata` structure and assign it to `wme->data`.
    - Copy the `cmd_find_state` structure from `fs` to `data->fs`.
    - Check if `args` is NULL or does not have the 'F' flag; if so, set `data->format` to the default format, otherwise set it to the format specified by the 'F' flag in `args`.
    - If the 'y' flag is present in `args`, set `data->prompt_flags` to `PROMPT_ACCEPT`.
    - Initialize a mode tree using `mode_tree_start`, passing various function pointers and data, and store the result in `data->data`.
    - Call `mode_tree_zoom` to adjust the mode tree based on `args`.
    - Build and draw the mode tree using `mode_tree_build` and `mode_tree_draw`.
    - Return the screen pointer `s` which is set by `mode_tree_start`.
- **Output**:
    - Returns a pointer to a `screen` structure, which represents the initialized screen for the customization mode.


---
### window_customize_destroy<!-- {{#callable:window_customize_destroy}} -->
The `window_customize_destroy` function deallocates memory and resources associated with a `window_customize_modedata` structure when its reference count reaches zero.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure that holds the data and resources to be freed.
- **Control Flow**:
    - Decrement the reference count of the `data` structure.
    - If the reference count is not zero, return immediately without freeing resources.
    - Iterate over each item in the `item_list` array and free each item using [`window_customize_free_item`](#window_customize_free_item).
    - Free the `item_list` array itself.
    - Free the `format` string associated with the `data` structure.
    - Free the `data` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided data structure.
- **Functions called**:
    - [`window_customize_free_item`](#window_customize_free_item)


---
### window_customize_free<!-- {{#callable:window_customize_free}} -->
The `window_customize_free` function releases resources associated with a window mode entry in a customization mode by marking the data as dead, freeing the mode tree, and destroying the mode data.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` which contains the mode data to be freed.
- **Control Flow**:
    - Retrieve the `window_customize_modedata` from the `wme` structure.
    - Check if the `data` is `NULL`, and if so, return immediately.
    - Set the `dead` flag of the `data` to 1 to indicate it is no longer active.
    - Call `mode_tree_free` to release the resources associated with the mode tree in `data`.
    - Call [`window_customize_destroy`](#window_customize_destroy) to clean up and free the `data` structure.
- **Output**:
    - This function does not return any value.
- **Functions called**:
    - [`window_customize_destroy`](#window_customize_destroy)


---
### window_customize_resize<!-- {{#callable:window_customize_resize}} -->
The `window_customize_resize` function resizes the mode tree data associated with a window mode entry to the specified dimensions.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure, which contains data related to a specific window mode.
    - `sx`: An unsigned integer representing the new width to resize to.
    - `sy`: An unsigned integer representing the new height to resize to.
- **Control Flow**:
    - Retrieve the `window_customize_modedata` structure from the `data` field of the `window_mode_entry` structure pointed to by `wme`.
    - Call the `mode_tree_resize` function with the `data` field of the `window_customize_modedata` structure and the new dimensions `sx` and `sy`.
- **Output**:
    - This function does not return a value; it performs an operation to resize the mode tree data.


---
### window_customize_free_callback<!-- {{#callable:window_customize_free_callback}} -->
The `window_customize_free_callback` function is responsible for freeing resources associated with a window customization mode data structure by calling the [`window_customize_destroy`](#window_customize_destroy) function.
- **Inputs**:
    - `modedata`: A pointer to the window customization mode data structure that needs to be freed.
- **Control Flow**:
    - The function takes a single argument, `modedata`, which is a pointer to the data structure to be freed.
    - It calls the [`window_customize_destroy`](#window_customize_destroy) function, passing `modedata` as an argument to handle the actual destruction and cleanup of resources.
- **Output**:
    - This function does not return any value; it performs a cleanup operation on the provided data structure.
- **Functions called**:
    - [`window_customize_destroy`](#window_customize_destroy)


---
### window_customize_free_item_callback<!-- {{#callable:window_customize_free_item_callback}} -->
The `window_customize_free_item_callback` function is responsible for freeing memory associated with a `window_customize_itemdata` structure and its associated `window_customize_modedata` structure.
- **Inputs**:
    - `itemdata`: A pointer to a `window_customize_itemdata` structure that needs to be freed.
- **Control Flow**:
    - Cast the `itemdata` pointer to a `window_customize_itemdata` structure pointer named `item`.
    - Retrieve the `data` field from the `item` structure, which is a pointer to a `window_customize_modedata` structure.
    - Call [`window_customize_free_item`](#window_customize_free_item) to free the memory associated with the `item`.
    - Call [`window_customize_destroy`](#window_customize_destroy) to free the memory associated with the `data` structure.
- **Output**:
    - The function does not return any value; it performs memory cleanup for the provided data structures.
- **Functions called**:
    - [`window_customize_free_item`](#window_customize_free_item)
    - [`window_customize_destroy`](#window_customize_destroy)


---
### window_customize_set_option_callback<!-- {{#callable:window_customize_set_option_callback}} -->
The `window_customize_set_option_callback` function sets a specified option for a window customization item, handling both array and non-array options, and updates the display accordingly.
- **Inputs**:
    - `c`: A pointer to the client structure, used for displaying status messages.
    - `itemdata`: A pointer to the window customization item data, which contains information about the option to be set.
    - `s`: A string representing the new value to set for the option.
    - `done`: An unused integer parameter, typically used to indicate completion status in callbacks.
- **Control Flow**:
    - Check if the input string `s` is NULL, empty, or if the data is marked as dead; if so, return 0 immediately.
    - Verify the validity of the item using [`window_customize_check_item`](#window_customize_check_item); if invalid, return 0.
    - Retrieve the options entry `o` using `options_get`; if not found, return 0.
    - Determine if the option is an array using [`options_table_entry`](tmux.h.driver.md#options_table_entry) and its flags.
    - If the option is an array and `idx` is -1, find the next available index in the array.
    - Set the array option value using `options_array_set`; if it fails, go to the fail label.
    - For non-array options, set the option value using `options_from_string`; if it fails, go to the fail label.
    - Push changes to the options and rebuild and redraw the mode tree.
    - Set the redraw flag for the window pane.
    - Return 0 on successful completion.
    - In the fail label, capitalize the first character of the error message, display it using `status_message_set`, free the error message, and return 0.
- **Output**:
    - Returns 0 to indicate the function's completion, regardless of success or failure.
- **Functions called**:
    - [`window_customize_check_item`](#window_customize_check_item)
    - [`options_table_entry`](tmux.h.driver.md#options_table_entry)


---
### window_customize_set_option<!-- {{#callable:window_customize_set_option}} -->
The `window_customize_set_option` function sets or modifies an option for a window, session, or pane in a customizable manner based on the provided parameters.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the option is being set.
    - `data`: A pointer to a `window_customize_modedata` structure containing mode-specific data for customization.
    - `item`: A pointer to a `window_customize_itemdata` structure representing the specific item (option) to be customized.
    - `global`: An integer flag indicating whether the option should be set globally (1) or locally (0).
    - `pane`: An integer flag indicating whether the option is specific to a pane (1) or not (0).
- **Control Flow**:
    - Check if the `item` is NULL or if the item is not valid using [`window_customize_check_item`](#window_customize_check_item); return if invalid.
    - Retrieve the current option entry `o` using `options_get` with the item's options object and name.
    - Determine the scope of the option based on the `global` and `pane` flags and the item's current scope.
    - If the option is a flag, toggle its value using `options_set_number`.
    - If the option is a choice, cycle through the available choices and set the next one using `options_set_number`.
    - For other types of options, prepare a prompt and value string for user input, and set up a status prompt using `status_prompt_set`.
- **Output**:
    - The function does not return a value; it modifies the state of the options and may trigger a user prompt for further input.
- **Functions called**:
    - [`window_customize_check_item`](#window_customize_check_item)
    - [`options_table_entry`](tmux.h.driver.md#options_table_entry)
    - [`window_customize_get_tree`](#window_customize_get_tree)
    - [`window_customize_scope_text`](#window_customize_scope_text)


---
### window_customize_unset_option<!-- {{#callable:window_customize_unset_option}} -->
The `window_customize_unset_option` function removes or resets an option entry for a given item in a window customization mode.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure, which contains the mode data for the window customization.
    - `item`: A pointer to a `window_customize_itemdata` structure, which represents the item whose option is to be unset or reset.
- **Control Flow**:
    - Check if the `item` is NULL or if the item is not valid for the given `data` using [`window_customize_check_item`](#window_customize_check_item); if so, return immediately.
    - Retrieve the options entry `o` using `options_get` with the item's options owner and name.
    - If the options entry `o` is NULL, return immediately.
    - If the item's index is not -1 and the item is the current item in the mode tree, move up the mode tree using `mode_tree_up`.
    - Call `options_remove_or_default` to remove or reset the option entry `o` at the specified index.
- **Output**:
    - The function does not return a value; it performs operations on the options entry associated with the given item.
- **Functions called**:
    - [`window_customize_check_item`](#window_customize_check_item)


---
### window_customize_reset_option<!-- {{#callable:window_customize_reset_option}} -->
The `window_customize_reset_option` function resets a specific option to its default value in a hierarchical options structure.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure, which contains the context for the customization mode.
    - `item`: A pointer to a `window_customize_itemdata` structure, representing the specific option item to be reset.
- **Control Flow**:
    - Check if the `item` is NULL or if it fails the [`window_customize_check_item`](#window_customize_check_item) validation; if so, return immediately.
    - Check if the `item` has a non-negative index (`idx`); if so, return immediately as it indicates an array element which should not be reset here.
    - Initialize `oo` to the options object associated with the `item`.
    - Iterate through the options hierarchy starting from `oo`, retrieving the specific option entry `o` by name.
    - If the option entry `o` is found, call `options_remove_or_default` to reset it to its default value.
    - Move to the parent options object and repeat until all levels are processed.
- **Output**:
    - The function does not return a value; it performs an in-place reset of the specified option to its default state.
- **Functions called**:
    - [`window_customize_check_item`](#window_customize_check_item)


---
### window_customize_set_command_callback<!-- {{#callable:window_customize_set_command_callback}} -->
The function `window_customize_set_command_callback` processes a command string for a key binding, updating the command list if the parsing is successful, or displaying an error message if it fails.
- **Inputs**:
    - `c`: A pointer to a `client` structure, representing the client context in which the command is being processed.
    - `itemdata`: A pointer to `window_customize_itemdata`, which contains data about the current item being customized, including its associated mode data.
    - `s`: A constant character pointer representing the command string to be parsed and set for the key binding.
    - `done`: An integer flag indicating whether the operation is complete, but it is unused in this function.
- **Control Flow**:
    - Check if the command string `s` is NULL, empty, or if the mode data is marked as dead; if so, return 0 immediately.
    - Retrieve the key binding associated with the item data; if it fails, return 0.
    - Parse the command string `s` using `cmd_parse_from_string`.
    - If parsing results in an error, convert the first character of the error message to uppercase, display it using `status_message_set`, free the error string, and return 0.
    - If parsing is successful, free the existing command list in the key binding and replace it with the new command list from the parse result.
    - Rebuild and redraw the mode tree using `mode_tree_build` and `mode_tree_draw`, and set the `PANE_REDRAW` flag on the window pane.
- **Output**:
    - The function returns an integer value, always 0, indicating the completion of the callback operation.
- **Functions called**:
    - [`window_customize_get_key`](#window_customize_get_key)


---
### window_customize_set_note_callback<!-- {{#callable:window_customize_set_note_callback}} -->
The function `window_customize_set_note_callback` updates the note of a key binding in a window customization mode if the provided string is valid and the data is not marked as dead.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `itemdata`: A pointer to `window_customize_itemdata`, which contains data about the item being customized.
    - `s`: A constant character pointer representing the new note string to be set for the key binding.
    - `done`: An integer indicating if the operation is done, which is unused in this function.
- **Control Flow**:
    - Check if the string `s` is NULL, empty, or if the data is marked as dead; if so, return 0.
    - Retrieve the `window_customize_itemdata` and `window_customize_modedata` from `itemdata`.
    - Check if `item` is NULL or if the key binding cannot be retrieved using [`window_customize_get_key`](#window_customize_get_key); if so, return 0.
    - Free the existing note of the key binding and duplicate the new note string `s` using `xstrdup`.
    - Rebuild and redraw the mode tree using `mode_tree_build` and `mode_tree_draw`.
    - Set the `PANE_REDRAW` flag on the window pane to indicate it needs to be redrawn.
    - Return 0 to indicate successful completion.
- **Output**:
    - The function returns an integer value of 0, indicating successful execution or early termination due to invalid input or state.
- **Functions called**:
    - [`window_customize_get_key`](#window_customize_get_key)


---
### window_customize_set_key<!-- {{#callable:window_customize_set_key}} -->
The `window_customize_set_key` function customizes key bindings for a window by setting or modifying key commands, notes, or repeat flags based on the current mode tree selection.
- **Inputs**:
    - `c`: A pointer to the client structure, representing the client for which the key customization is being set.
    - `data`: A pointer to the `window_customize_modedata` structure, which contains the mode data for the window customization process.
    - `item`: A pointer to the `window_customize_itemdata` structure, representing the specific item (key) being customized.
- **Control Flow**:
    - Check if the `item` is NULL or if the key binding cannot be retrieved using [`window_customize_get_key`](#window_customize_get_key); if so, return immediately.
    - Retrieve the current name from the mode tree data using `mode_tree_get_current_name`.
    - If the current name is 'Repeat', toggle the `KEY_BINDING_REPEAT` flag for the key binding.
    - If the current name is 'Command', prepare a prompt and value for the command associated with the key, create a new item data structure, and set a status prompt for the client to modify the command.
    - If the current name is 'Note', prepare a prompt for the note associated with the key, create a new item data structure, and set a status prompt for the client to modify the note.
- **Output**:
    - The function does not return a value; it modifies the key binding associated with the given item based on the current mode tree selection.
- **Functions called**:
    - [`window_customize_get_key`](#window_customize_get_key)


---
### window_customize_unset_key<!-- {{#callable:window_customize_unset_key}} -->
The `window_customize_unset_key` function removes a key binding from a key table if the specified item is valid and currently selected.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure, which contains the mode data for the window customization.
    - `item`: A pointer to a `window_customize_itemdata` structure, which represents the item data for the key to be unset.
- **Control Flow**:
    - Check if the `item` is NULL or if the key cannot be retrieved using [`window_customize_get_key`](#window_customize_get_key); if so, return immediately.
    - If the `item` is the current item in the mode tree, collapse the current mode tree item and move up in the mode tree.
    - Remove the key binding from the key table using `key_bindings_remove` with the key table's name and the key from the binding.
- **Output**:
    - This function does not return a value; it performs operations to modify the key bindings.
- **Functions called**:
    - [`window_customize_get_key`](#window_customize_get_key)


---
### window_customize_reset_key<!-- {{#callable:window_customize_reset_key}} -->
The `window_customize_reset_key` function resets a key binding to its default state in a specified key table if necessary.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure, which contains the mode data for the window customization.
    - `item`: A pointer to a `window_customize_itemdata` structure, which represents the item data for the key to be reset.
- **Control Flow**:
    - Check if the `item` is NULL or if the key table and binding cannot be retrieved using [`window_customize_get_key`](#window_customize_get_key); if so, return immediately.
    - Retrieve the default key binding `dd` for the specified key in the key table `kt`.
    - If the default key binding `dd` is not NULL and the current binding's command list matches the default, return without making changes.
    - If the default key binding `dd` is NULL and the `item` is the current item in the mode tree, collapse the current mode tree item and move up in the mode tree.
    - Reset the key binding in the key table `kt` to its default state using `key_bindings_reset`.
- **Output**:
    - The function does not return a value; it performs operations to reset a key binding to its default state if applicable.
- **Functions called**:
    - [`window_customize_get_key`](#window_customize_get_key)


---
### window_customize_change_each<!-- {{#callable:window_customize_change_each}} -->
The `window_customize_change_each` function modifies a window customization item based on its scope and the specified change type (unset or reset).
- **Inputs**:
    - `modedata`: A pointer to `window_customize_modedata` structure containing mode-specific data.
    - `itemdata`: A pointer to `window_customize_itemdata` structure representing the item to be changed.
    - `c`: Unused parameter, typically a pointer to a `client` structure.
    - `key`: Unused parameter, typically a key code.
- **Control Flow**:
    - Cast `modedata` to `window_customize_modedata` and `itemdata` to `window_customize_itemdata`.
    - Check the `change` field of `data` to determine the type of change (unset or reset).
    - If the change is `WINDOW_CUSTOMIZE_UNSET`, call [`window_customize_unset_key`](#window_customize_unset_key) or [`window_customize_unset_option`](#window_customize_unset_option) based on the item's scope.
    - If the change is `WINDOW_CUSTOMIZE_RESET`, call [`window_customize_reset_key`](#window_customize_reset_key) or [`window_customize_reset_option`](#window_customize_reset_option) based on the item's scope.
    - If the item's scope is not `WINDOW_CUSTOMIZE_KEY`, call `options_push_changes` with the item's name.
- **Output**:
    - The function does not return a value; it performs actions on the provided item based on its scope and the specified change type.
- **Functions called**:
    - [`window_customize_unset_key`](#window_customize_unset_key)
    - [`window_customize_unset_option`](#window_customize_unset_option)
    - [`window_customize_reset_key`](#window_customize_reset_key)
    - [`window_customize_reset_option`](#window_customize_reset_option)


---
### window_customize_change_current_callback<!-- {{#callable:window_customize_change_current_callback}} -->
The function `window_customize_change_current_callback` processes a confirmation input to either unset or reset a customization item in a window mode, and updates the display accordingly.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `modedata`: A pointer to a `window_customize_modedata` structure, which contains the mode data for the window customization.
    - `s`: A string representing the user's input, expected to be a confirmation ('y' or 'Y') to proceed with the change.
    - `done`: An integer indicating if the operation is done, which is unused in this function.
- **Control Flow**:
    - Check if the input string `s` is NULL, empty, or if the `data` is marked as dead; if so, return 0 immediately.
    - Convert the first character of `s` to lowercase and check if it is 'y' and the string length is 1; if not, return 0.
    - Retrieve the current item from the mode tree using `mode_tree_get_current`.
    - Switch on the `change` field of `data` to determine whether to unset or reset the current item.
    - If the item's scope is not `WINDOW_CUSTOMIZE_KEY`, call `options_push_changes` with the item's name to apply changes.
    - Rebuild and redraw the mode tree using `mode_tree_build` and `mode_tree_draw`.
    - Set the `PANE_REDRAW` flag on the window pane to indicate that the pane needs to be redrawn.
- **Output**:
    - The function returns an integer 0, indicating the operation's completion status.
- **Functions called**:
    - [`window_customize_unset_key`](#window_customize_unset_key)
    - [`window_customize_unset_option`](#window_customize_unset_option)
    - [`window_customize_reset_key`](#window_customize_reset_key)
    - [`window_customize_reset_option`](#window_customize_reset_option)


---
### window_customize_change_tagged_callback<!-- {{#callable:window_customize_change_tagged_callback}} -->
The `window_customize_change_tagged_callback` function processes a confirmation input to apply changes to all tagged items in a mode tree, rebuilding and redrawing the tree if confirmed.
- **Inputs**:
    - `c`: A pointer to a `client` structure, representing the client that initiated the callback.
    - `modedata`: A pointer to `window_customize_modedata`, which contains the mode data for the window customization.
    - `s`: A constant character pointer representing the input string, typically a confirmation response.
    - `done`: An integer flag indicating if the operation is completed, but it is unused in this function.
- **Control Flow**:
    - Check if the input string `s` is NULL, empty, or if the `data->dead` flag is set; if any of these conditions are true, return 0 immediately.
    - Convert the first character of `s` to lowercase and check if it is 'y' and if `s` has no other characters; if not, return 0.
    - Call `mode_tree_each_tagged` to apply the `window_customize_change_each` function to each tagged item in the mode tree, passing the client `c` and other parameters.
    - Rebuild the mode tree using `mode_tree_build`.
    - Redraw the mode tree using `mode_tree_draw`.
    - Set the `PANE_REDRAW` flag on `data->wp` to indicate that the pane needs to be redrawn.
    - Return 0.
- **Output**:
    - The function returns an integer value of 0, indicating the operation's completion or early exit.


---
### window_customize_key<!-- {{#callable:window_customize_key}} -->
The `window_customize_key` function handles key events for customizing window options and key bindings in a mode tree interface.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode entry.
    - `c`: A pointer to the `client` structure representing the client interacting with the window.
    - `s`: Unused parameter, a pointer to the `session` structure.
    - `wl`: Unused parameter, a pointer to the `winlink` structure.
    - `key`: The key code representing the key event that triggered the function.
    - `m`: A pointer to the `mouse_event` structure representing any mouse event associated with the key event.
- **Control Flow**:
    - Retrieve the current item from the mode tree using `mode_tree_get_current` and store it in `item`.
    - Process the key event using `mode_tree_key`, updating the `finished` status and potentially changing the current item.
    - If the key is '\r' or 's', set the key or option based on the item's scope and rebuild the mode tree.
    - If the key is 'w', set the option without global scope and rebuild the mode tree.
    - If the key is 'S' or 'W', set the option with global scope and rebuild the mode tree.
    - If the key is 'd', prompt the user to reset the current item to default if applicable.
    - If the key is 'D', prompt the user to reset all tagged items to default if applicable.
    - If the key is 'u', prompt the user to unset the current item if applicable.
    - If the key is 'U', prompt the user to unset all tagged items if applicable.
    - If the key is 'H', toggle the visibility of global options and rebuild the mode tree.
    - If the `finished` flag is set, reset the window pane mode; otherwise, redraw the mode tree and set the pane redraw flag.
- **Output**:
    - The function does not return a value; it modifies the state of the mode tree and window pane based on the key event.
- **Functions called**:
    - [`window_customize_set_key`](#window_customize_set_key)
    - [`window_customize_set_option`](#window_customize_set_option)


# Function Declarations (Public API)

---
### window_customize_init<!-- {{#callable_declaration:window_customize_init}} -->
Initialize a screen for customizing window options.
- **Description**: This function sets up a screen for customizing window options within a specified window mode entry. It should be called when you need to initialize the customization interface for a window pane. The function requires a valid window mode entry and command find state. It optionally accepts arguments to customize the format and prompt behavior. The function handles memory allocation for internal data structures and initializes the mode tree for displaying options.
- **Inputs**:
    - `wme`: A pointer to a window_mode_entry structure representing the window mode entry to be customized. Must not be null.
    - `fs`: A pointer to a cmd_find_state structure representing the command find state. Must not be null.
    - `args`: A pointer to an args structure containing optional arguments for customization. Can be null, in which case default settings are used.
- **Output**: Returns a pointer to a screen structure representing the initialized customization screen.
- **See also**: [`window_customize_init`](#window_customize_init)  (Implementation)


---
### window_customize_free<!-- {{#callable_declaration:window_customize_free}} -->
Releases resources associated with a window mode entry.
- **Description**: This function is used to clean up and release resources associated with a window mode entry when it is no longer needed. It should be called to ensure that any allocated memory or resources are properly freed, preventing memory leaks. This function is typically called when a window mode is being closed or switched out. It is important to ensure that the window mode entry is valid and has been initialized before calling this function.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the window mode entry to be freed. This parameter must not be null, and it is expected that the caller has ownership of the memory pointed to by this parameter. If the `data` field of `wme` is null, the function will return immediately without performing any operations.
- **Output**: None
- **See also**: [`window_customize_free`](#window_customize_free)  (Implementation)


---
### window_customize_resize<!-- {{#callable_declaration:window_customize_resize}} -->
Resizes the window customization mode.
- **Description**: This function adjusts the size of the window customization mode to fit the specified dimensions. It should be called whenever the window size changes to ensure that the customization interface is properly displayed. This function does not perform any operations if the provided dimensions are invalid.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` representing the current window mode entry. Must not be null, and the `data` field should be properly initialized.
    - `sx`: An unsigned integer representing the new width of the window. Must be a valid non-negative value.
    - `sy`: An unsigned integer representing the new height of the window. Must be a valid non-negative value.
- **Output**: None
- **See also**: [`window_customize_resize`](#window_customize_resize)  (Implementation)


---
### window_customize_key<!-- {{#callable_declaration:window_customize_key}} -->
Handles key events for customizing window options and key bindings.
- **Description**: This function processes key events to customize window options and key bindings within a window mode entry. It should be called when a key event occurs in the context of window customization. The function supports various key commands to set, unset, or reset options and key bindings, and it updates the display accordingly. It requires a valid window mode entry and client, and it assumes that the session and winlink parameters are not used.
- **Inputs**:
    - `wme`: A pointer to a window_mode_entry structure representing the current mode entry. Must not be null.
    - `c`: A pointer to a client structure representing the client that triggered the key event. Must not be null.
    - `s`: Unused parameter, should be ignored.
    - `wl`: Unused parameter, should be ignored.
    - `key`: A key_code representing the key event to be processed. Valid key codes are expected.
    - `m`: A pointer to a mouse_event structure representing any associated mouse event. Can be null if no mouse event is associated.
- **Output**: None
- **See also**: [`window_customize_key`](#window_customize_key)  (Implementation)


