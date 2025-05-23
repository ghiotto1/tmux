# Purpose
This C source code file is part of the `tmux` project, specifically implementing a window mode for customizing options and key bindings within the `tmux` terminal multiplexer. The file defines a mode called "options-mode," which allows users to interactively view and modify various configuration options and key bindings associated with `tmux` sessions, windows, and panes. The code provides a structured interface for displaying options in a tree-like format, enabling users to navigate, select, and edit options or key bindings through a menu-driven interface.

Key components of the code include the definition of the `window_customize_mode` structure, which specifies the mode's name, default format, and associated functions for initialization, resizing, key handling, and cleanup. The code also defines several enumerations and data structures to manage the state and data of the customization mode, such as `window_customize_scope`, `window_customize_change`, and `window_customize_modedata`. The file implements functions to build and display the options and key bindings tree, handle user input, and apply changes to the `tmux` configuration. This code is intended to be integrated into the `tmux` application, providing a user interface for customizing its behavior without directly editing configuration files.
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
- **Type**: `struct screen*`
- **Description**: The `window_customize_init` variable is a static function that returns a pointer to a `struct screen`. It is responsible for initializing the customization mode for a window in the tmux application. This function sets up the necessary data structures and configurations to allow users to customize window options and key bindings.
- **Use**: This variable is used to initialize the customization mode for a window, setting up the screen and related data structures for user interaction.


---
### window_customize_menu_items
- **Type**: ``static const struct menu_item[]``
- **Description**: The `window_customize_menu_items` is a static constant array of `struct menu_item` that defines a set of menu items for a window customization interface. Each element in the array represents a menu item with a label, a key code, and a NULL pointer for additional data. The array is terminated with a NULL entry to indicate the end of the menu items.
- **Use**: This variable is used to define the menu options available in the window customization mode, allowing users to interact with the interface using specific key bindings.


---
### window_customize_mode
- **Type**: `const struct window_mode`
- **Description**: The `window_customize_mode` is a constant structure of type `window_mode` that defines a specific mode for customizing window options in the application. It includes a name, a default format for displaying options, and function pointers for initializing, freeing, resizing, and handling key events in this mode.
- **Use**: This variable is used to define and manage the behavior of the 'options-mode' in the application, allowing users to customize window options.


# Data Structures

---
### window_customize_scope
- **Type**: `enum`
- **Members**:
    - `WINDOW_CUSTOMIZE_NONE`: Represents no customization scope.
    - `WINDOW_CUSTOMIZE_KEY`: Represents customization scope for keys.
    - `WINDOW_CUSTOMIZE_SERVER`: Represents customization scope for the server.
    - `WINDOW_CUSTOMIZE_GLOBAL_SESSION`: Represents customization scope for global session options.
    - `WINDOW_CUSTOMIZE_SESSION`: Represents customization scope for session options.
    - `WINDOW_CUSTOMIZE_GLOBAL_WINDOW`: Represents customization scope for global window options.
    - `WINDOW_CUSTOMIZE_WINDOW`: Represents customization scope for window options.
    - `WINDOW_CUSTOMIZE_PANE`: Represents customization scope for pane options.
- **Description**: The `window_customize_scope` enumeration defines various levels of customization scopes within a window management system, such as server, session, window, and pane levels. Each enumerator represents a specific scope where customization can be applied, allowing for fine-grained control over different aspects of the window management environment.


---
### window_customize_change
- **Type**: `enum`
- **Members**:
    - `WINDOW_CUSTOMIZE_UNSET`: Represents an unset state for window customization.
    - `WINDOW_CUSTOMIZE_RESET`: Represents a reset state for window customization.
- **Description**: The `window_customize_change` is an enumeration that defines possible states for customizing a window in the context of the software. It includes two states: `WINDOW_CUSTOMIZE_UNSET`, which indicates that no customization has been set, and `WINDOW_CUSTOMIZE_RESET`, which indicates that the customization should be reset to its default state. This enumeration is likely used to manage and track the customization state of windows within the application.


---
### window_customize_itemdata
- **Type**: `struct`
- **Members**:
    - `data`: A pointer to a `window_customize_modedata` structure, representing the mode data associated with the item.
    - `scope`: An enumeration value of type `window_customize_scope` indicating the scope of customization.
    - `table`: A pointer to a character string representing the name of the key table.
    - `key`: A `key_code` representing the key associated with the item.
    - `oo`: A pointer to an `options` structure, representing the options associated with the item.
    - `name`: A pointer to a character string representing the name of the item.
    - `idx`: An integer representing the index of the item, used for array options.
- **Description**: The `window_customize_itemdata` structure is used to represent an individual item in the window customization mode of a terminal multiplexer. It holds information about the item's associated mode data, scope, key table, key code, options, name, and index. This structure is part of a larger system that allows users to customize window options and key bindings within the terminal multiplexer environment.


---
### window_customize_modedata
- **Type**: `struct`
- **Members**:
    - `wp`: A pointer to a `window_pane` structure, representing the pane associated with this mode data.
    - `dead`: An integer flag indicating if the mode data is no longer active or valid.
    - `references`: An integer count of references to this mode data, used for memory management.
    - `data`: A pointer to a `mode_tree_data` structure, which holds the data for the mode tree.
    - `format`: A string representing the format used for displaying options in the mode.
    - `hide_global`: An integer flag indicating whether global options should be hidden.
    - `prompt_flags`: An integer representing flags for prompt behavior.
    - `item_list`: A pointer to an array of pointers to `window_customize_itemdata` structures, representing the items in the mode.
    - `item_size`: An unsigned integer representing the number of items in the `item_list`.
    - `fs`: A `cmd_find_state` structure representing the state of command finding.
    - `change`: An enum `window_customize_change` indicating the type of change to be applied.
- **Description**: The `window_customize_modedata` structure is used in the context of customizing window options in a terminal multiplexer environment. It holds various data related to the customization mode, including a reference to the associated window pane, a mode tree for displaying options, and a list of items representing customizable options. The structure also includes flags and format strings for controlling the display and behavior of the customization interface. It is designed to manage the state and interactions within a window customization mode, allowing users to view and modify window and pane options.


# Functions

---
### window_customize_get_tag<!-- {{#callable:window_customize_get_tag}} -->
The function `window_customize_get_tag` generates a unique 64-bit tag for a given options entry, index, and options table entry.
- **Inputs**:
    - `o`: A pointer to an `options_entry` structure, representing the options entry for which the tag is being generated.
    - `idx`: An integer representing the index of the options entry, used in the tag generation.
    - `oe`: A pointer to an `options_table_entry` structure, representing the options table entry associated with the options entry, or NULL if not applicable.
- **Control Flow**:
    - Check if `oe` is NULL; if so, return the casted pointer of `o` as a 64-bit unsigned integer.
    - Calculate the offset of `oe` from the start of the `options_table` by subtracting the base address of `options_table` from `oe` and dividing by the size of an `options_table` entry.
    - Construct a 64-bit tag by combining a fixed bit pattern, the calculated offset, the incremented index, and a fixed bit pattern using bitwise operations.
- **Output**:
    - A 64-bit unsigned integer representing a unique tag for the given options entry and index.


---
### window_customize_get_tree<!-- {{#callable:window_customize_get_tree}} -->
The `window_customize_get_tree` function returns a pointer to the options structure corresponding to a given customization scope and command find state.
- **Inputs**:
    - `scope`: An enum value of type `window_customize_scope` that specifies the scope of customization (e.g., server, session, window, pane, etc.).
    - `fs`: A pointer to a `cmd_find_state` structure that contains the current state of command finding, including session, window, and pane information.
- **Control Flow**:
    - The function uses a switch statement to determine the action based on the `scope` value.
    - If the `scope` is `WINDOW_CUSTOMIZE_NONE` or `WINDOW_CUSTOMIZE_KEY`, the function returns `NULL`.
    - If the `scope` is `WINDOW_CUSTOMIZE_SERVER`, it returns `global_options`.
    - If the `scope` is `WINDOW_CUSTOMIZE_GLOBAL_SESSION`, it returns `global_s_options`.
    - If the `scope` is `WINDOW_CUSTOMIZE_SESSION`, it returns the options of the session in `fs`.
    - If the `scope` is `WINDOW_CUSTOMIZE_GLOBAL_WINDOW`, it returns `global_w_options`.
    - If the `scope` is `WINDOW_CUSTOMIZE_WINDOW`, it returns the options of the window in `fs`.
    - If the `scope` is `WINDOW_CUSTOMIZE_PANE`, it returns the options of the pane in `fs`.
    - If none of the cases match, it returns `NULL`.
- **Output**:
    - A pointer to a `struct options` corresponding to the specified scope, or `NULL` if the scope is not applicable.


---
### window_customize_check_item<!-- {{#callable:window_customize_check_item}} -->
The function `window_customize_check_item` checks if a given item in a window customization mode matches the current state of options based on a specified scope.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure containing the current mode data, including the find state and window pane.
    - `item`: A pointer to a `window_customize_itemdata` structure representing the item to be checked, including its scope and options.
    - `fsp`: A pointer to a `cmd_find_state` structure, which may be NULL, used to determine the current find state for the operation.
- **Control Flow**:
    - A local `cmd_find_state` structure `fs` is declared.
    - If `fsp` is NULL, it is set to point to the local `fs`.
    - The function checks if the find state in `data` is valid using [`cmd_find_valid_state`](cmd-find.c.driver.md#cmd_find_valid_state).
    - If valid, it copies the find state from `data` to `fsp` using [`cmd_find_copy_state`](cmd-find.c.driver.md#cmd_find_copy_state).
    - If not valid, it initializes `fsp` from the pane in `data` using [`cmd_find_from_pane`](cmd-find.c.driver.md#cmd_find_from_pane).
    - The function returns the result of comparing the options owner of `item` with the options tree obtained from [`window_customize_get_tree`](#window_customize_get_tree) using `item`'s scope and `fsp`.
- **Output**:
    - The function returns an integer indicating whether the options owner of the item matches the options tree for the given scope and find state.
- **Functions called**:
    - [`cmd_find_valid_state`](cmd-find.c.driver.md#cmd_find_valid_state)
    - [`cmd_find_copy_state`](cmd-find.c.driver.md#cmd_find_copy_state)
    - [`cmd_find_from_pane`](cmd-find.c.driver.md#cmd_find_from_pane)
    - [`window_customize_get_tree`](#window_customize_get_tree)


---
### window_customize_get_key<!-- {{#callable:window_customize_get_key}} -->
The `window_customize_get_key` function retrieves a key binding from a specified key table and optionally returns pointers to the key table and key binding if they are found.
- **Inputs**:
    - `item`: A pointer to a `window_customize_itemdata` structure that contains the key table name and key code to search for.
    - `ktp`: A pointer to a pointer to a `key_table` structure where the found key table will be stored, if not NULL.
    - `bdp`: A pointer to a pointer to a `key_binding` structure where the found key binding will be stored, if not NULL.
- **Control Flow**:
    - Retrieve the key table using [`key_bindings_get_table`](key-bindings.c.driver.md#key_bindings_get_table) with the table name from `item` and a flag of 0.
    - If the key table is not found, return 0.
    - Retrieve the key binding from the key table using [`key_bindings_get`](key-bindings.c.driver.md#key_bindings_get) with the key from `item`.
    - If the key binding is not found, return 0.
    - If `ktp` is not NULL, store the found key table in `*ktp`.
    - If `bdp` is not NULL, store the found key binding in `*bdp`.
    - Return 1 to indicate success.
- **Output**:
    - Returns 1 if the key table and key binding are found, otherwise returns 0.
- **Functions called**:
    - [`key_bindings_get_table`](key-bindings.c.driver.md#key_bindings_get_table)
    - [`key_bindings_get`](key-bindings.c.driver.md#key_bindings_get)


---
### window_customize_scope_text<!-- {{#callable:window_customize_scope_text}} -->
The `window_customize_scope_text` function generates a descriptive string based on the given customization scope and command find state.
- **Inputs**:
    - `scope`: An enumeration value of type `enum window_customize_scope` that specifies the scope of customization (e.g., pane, session, window).
    - `fs`: A pointer to a `struct cmd_find_state` which contains the current state of command finding, including references to session, window, and pane.
- **Control Flow**:
    - The function starts by declaring a character pointer `s` and an unsigned integer `idx`.
    - A switch statement is used to determine the behavior based on the `scope` argument.
    - If the scope is `WINDOW_CUSTOMIZE_PANE`, it retrieves the pane index using [`window_pane_index`](window.c.driver.md#window_pane_index) and formats the string as 'pane <index>'.
    - If the scope is `WINDOW_CUSTOMIZE_SESSION`, it formats the string as 'session <session_name>'.
    - If the scope is `WINDOW_CUSTOMIZE_WINDOW`, it formats the string as 'window <window_index>'.
    - For any other scope, it defaults to an empty string.
    - The function returns the formatted string `s`.
- **Output**:
    - The function returns a dynamically allocated string that describes the scope, such as 'pane <index>', 'session <name>', or 'window <index>', or an empty string if the scope is not recognized.
- **Functions called**:
    - [`window_pane_index`](window.c.driver.md#window_pane_index)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### window_customize_add_item<!-- {{#callable:window_customize_add_item}} -->
The function `window_customize_add_item` adds a new item to the `item_list` of a `window_customize_modedata` structure, reallocating memory as necessary.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure, which contains the `item_list` and its current size (`item_size`).
- **Control Flow**:
    - Reallocate the `item_list` array to accommodate one more `window_customize_itemdata` structure using [`xreallocarray`](xmalloc.c.driver.md#xreallocarray).
    - Allocate memory for a new `window_customize_itemdata` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc) and assign it to the newly added position in the `item_list`.
    - Increment the `item_size` to reflect the addition of the new item.
    - Return a pointer to the newly added `window_customize_itemdata` structure.
- **Output**:
    - A pointer to the newly added `window_customize_itemdata` structure in the `item_list`.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


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
    - `ft`: A pointer to a `format_tree` structure used for formatting the options' names and values.
- **Control Flow**:
    - Retrieve the options table entry and owner for the given options entry `o`.
    - Initialize the first item in the options array using [`options_array_first`](options.c.driver.md#options_array_first).
    - Iterate over each item in the options array using a while loop.
    - For each item, retrieve its index and format its name and value using [`xasprintf`](xmalloc.c.driver.md#xasprintf) and [`options_to_string`](options.c.driver.md#options_to_string).
    - Add the formatted name and value to the format tree `ft`.
    - Create a new `window_customize_itemdata` item and set its properties, including scope, owner, name, and index.
    - Expand the format tree to generate a text representation of the item.
    - Calculate a unique tag for the item using [`window_customize_get_tag`](#window_customize_get_tag).
    - Add the item to the mode tree using [`mode_tree_add`](mode-tree.c.driver.md#mode_tree_add).
    - Free the allocated memory for the formatted name, value, and expanded text.
    - Move to the next item in the options array using [`options_array_next`](options.c.driver.md#options_array_next).
- **Output**:
    - The function does not return a value; it modifies the mode tree by adding formatted options items to it.
- **Functions called**:
    - [`options_table_entry`](tmux.h.driver.md#options_table_entry)
    - [`options_owner`](options.c.driver.md#options_owner)
    - [`options_array_first`](options.c.driver.md#options_array_first)
    - [`options_array_item_index`](options.c.driver.md#options_array_item_index)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`options_name`](options.c.driver.md#options_name)
    - [`options_to_string`](options.c.driver.md#options_to_string)
    - [`window_customize_add_item`](#window_customize_add_item)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`window_customize_get_tag`](#window_customize_get_tag)
    - [`mode_tree_add`](mode-tree.c.driver.md#mode_tree_add)
    - [`options_array_next`](options.c.driver.md#options_array_next)


---
### window_customize_build_option<!-- {{#callable:window_customize_build_option}} -->
The `window_customize_build_option` function constructs and formats a mode tree item for a given option, considering its scope, type, and any applied filters.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure that holds the customization mode data.
    - `top`: A pointer to a `mode_tree_item` structure representing the top of the mode tree where the option will be added.
    - `scope`: An enum value of type `window_customize_scope` indicating the scope of the option (e.g., server, session, window).
    - `o`: A pointer to an `options_entry` structure representing the option to be customized.
    - `ft`: A pointer to a `format_tree` structure used for formatting the option's display.
    - `filter`: A string representing a filter to apply to the option, or NULL if no filter is applied.
    - `fs`: A pointer to a `cmd_find_state` structure used to find the current command state.
- **Control Flow**:
    - Retrieve the options table entry and owner for the given option `o`.
    - Check if the option is a hook or an array and set flags accordingly.
    - Determine if the option is global based on its scope and the `hide_global` flag in `data`.
    - Add formatted information about the option (name, global status, array status, scope, unit) to the format tree `ft`.
    - If the option is not an array, convert its value to a string and add it to the format tree.
    - If a filter is provided, expand it using the format tree and check if it evaluates to true; if not, return early.
    - Create a new `window_customize_itemdata` item, set its properties, and add it to the mode tree.
    - If the option is an array, call [`window_customize_build_array`](#window_customize_build_array) to handle array elements.
- **Output**:
    - The function does not return a value but modifies the mode tree by adding a new item representing the option, formatted according to its properties and any applied filters.
- **Functions called**:
    - [`options_table_entry`](tmux.h.driver.md#options_table_entry)
    - [`options_owner`](options.c.driver.md#options_owner)
    - [`options_name`](options.c.driver.md#options_name)
    - [`window_customize_scope_text`](#window_customize_scope_text)
    - [`options_to_string`](options.c.driver.md#options_to_string)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`format_true`](format.c.driver.md#format_true)
    - [`window_customize_add_item`](#window_customize_add_item)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`window_customize_get_tag`](#window_customize_get_tag)
    - [`mode_tree_add`](mode-tree.c.driver.md#mode_tree_add)
    - [`window_customize_build_array`](#window_customize_build_array)


---
### window_customize_find_user_options<!-- {{#callable:window_customize_find_user_options}} -->
The function `window_customize_find_user_options` identifies and collects user-defined options from a given options tree, adding them to a list if they are not already present.
- **Inputs**:
    - `oo`: A pointer to a struct `options`, representing the options tree to search through.
    - `list`: A pointer to a list of strings (const char ***) that will be updated to include new user-defined options found.
    - `size`: A pointer to an unsigned integer (u_int) representing the current size of the list, which will be updated as new options are added.
- **Control Flow**:
    - Initialize a pointer `o` to the first entry in the options tree `oo` using [`options_first`](options.c.driver.md#options_first).
    - Enter a loop that continues as long as `o` is not NULL.
    - Retrieve the name of the current option entry using [`options_name`](options.c.driver.md#options_name).
    - Check if the option name starts with '@'; if not, move to the next option using [`options_next`](options.c.driver.md#options_next) and continue the loop.
    - Iterate over the existing list to check if the current option name is already present.
    - If the option name is found in the list, move to the next option using [`options_next`](options.c.driver.md#options_next) and continue the loop.
    - If the option name is not found in the list, reallocate the list to accommodate one more entry using [`xreallocarray`](xmalloc.c.driver.md#xreallocarray).
    - Add the new option name to the list and increment the size counter.
    - Move to the next option using [`options_next`](options.c.driver.md#options_next).
- **Output**:
    - The function updates the list and size to include any new user-defined options found in the options tree `oo` that start with '@' and are not already in the list.
- **Functions called**:
    - [`options_first`](options.c.driver.md#options_first)
    - [`options_name`](options.c.driver.md#options_name)
    - [`options_next`](options.c.driver.md#options_next)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)


---
### window_customize_build_options<!-- {{#callable:window_customize_build_options}} -->
The `window_customize_build_options` function constructs a mode tree of customizable options from multiple option trees, applying filters and formatting as specified.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure that holds the mode data for customization.
    - `title`: A string representing the title of the mode tree.
    - `tag`: A 64-bit unsigned integer used as a tag for the mode tree item.
    - `scope0`: An enum value of type `window_customize_scope` representing the scope of the first options tree.
    - `oo0`: A pointer to the first `options` structure from which options are retrieved.
    - `scope1`: An enum value of type `window_customize_scope` representing the scope of the second options tree.
    - `oo1`: A pointer to the second `options` structure, which may be NULL.
    - `scope2`: An enum value of type `window_customize_scope` representing the scope of the third options tree.
    - `oo2`: A pointer to the third `options` structure, which may be NULL.
    - `ft`: A pointer to a `format_tree` structure used for formatting options.
    - `filter`: A string representing a filter to apply to the options, which may be NULL.
    - `fs`: A pointer to a `cmd_find_state` structure used for command state information.
- **Control Flow**:
    - Initialize a top-level mode tree item with the given title and tag.
    - Find user-defined options from the first options tree and append them to a list.
    - If the second and third options trees are not NULL, find user-defined options from them and append to the list.
    - Iterate over the list of user-defined options, retrieving each option from the third, second, or first options tree in that order.
    - Determine the scope of each option based on its owner and call [`window_customize_build_option`](#window_customize_build_option) to add it to the mode tree.
    - Free the list of user-defined options.
    - Iterate over all options in the first options tree, skipping those starting with '@'.
    - For each option, retrieve it from the third, second, or first options tree in that order, determine its scope, and call [`window_customize_build_option`](#window_customize_build_option) to add it to the mode tree.
- **Output**:
    - The function does not return a value; it modifies the mode tree within the `window_customize_modedata` structure to include the options from the specified trees.
- **Functions called**:
    - [`mode_tree_add`](mode-tree.c.driver.md#mode_tree_add)
    - [`mode_tree_no_tag`](mode-tree.c.driver.md#mode_tree_no_tag)
    - [`window_customize_find_user_options`](#window_customize_find_user_options)
    - [`options_get`](options.c.driver.md#options_get)
    - [`options_owner`](options.c.driver.md#options_owner)
    - [`window_customize_build_option`](#window_customize_build_option)
    - [`options_first`](options.c.driver.md#options_first)
    - [`options_name`](options.c.driver.md#options_name)
    - [`options_next`](options.c.driver.md#options_next)


---
### window_customize_build_keys<!-- {{#callable:window_customize_build_keys}} -->
The `window_customize_build_keys` function constructs a hierarchical mode tree of key bindings from a given key table, applying optional filters and formatting, and adds them to a mode tree data structure for display or manipulation.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure that holds the mode data and state for customization.
    - `kt`: A pointer to a `key_table` structure representing the key table to be processed.
    - `ft`: A pointer to a `format_tree` structure used for formatting the output.
    - `filter`: A constant character pointer to a filter string used to determine which key bindings to include.
    - `fs`: A pointer to a `cmd_find_state` structure used to find the current command state.
    - `number`: An unsigned integer used to generate a unique tag for the mode tree items.
- **Control Flow**:
    - Generate a unique tag using the provided number and bitwise operations.
    - Create a title string for the key table and add a top-level item to the mode tree with this title.
    - Initialize a format tree from the command find state and add key-related format variables.
    - Iterate over each key binding in the key table.
    - For each key binding, add its key and note (if present) to the format tree.
    - If a filter is provided, expand it using the format tree and check if it evaluates to true; if not, skip the current key binding.
    - Add a new item to the mode tree for the key binding, including its command list and note as child items.
    - Determine if the key binding has a repeat flag and add this information as a child item.
    - Move to the next key binding in the table.
    - Free the format tree after processing all key bindings.
- **Output**:
    - The function does not return a value; it modifies the mode tree data structure by adding items representing key bindings and their associated information.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`mode_tree_add`](mode-tree.c.driver.md#mode_tree_add)
    - [`mode_tree_no_tag`](mode-tree.c.driver.md#mode_tree_no_tag)
    - [`format_create_from_state`](format.c.driver.md#format_create_from_state)
    - [`key_bindings_first`](key-bindings.c.driver.md#key_bindings_first)
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`format_true`](format.c.driver.md#format_true)
    - [`window_customize_add_item`](#window_customize_add_item)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`cmd_list_print`](cmd.c.driver.md#cmd_list_print)
    - [`mode_tree_draw_as_parent`](mode-tree.c.driver.md#mode_tree_draw_as_parent)
    - [`key_bindings_next`](key-bindings.c.driver.md#key_bindings_next)
    - [`format_free`](format.c.driver.md#format_free)


---
### window_customize_build<!-- {{#callable:window_customize_build}} -->
The `window_customize_build` function initializes and populates a data structure with options and key bindings for a window customization mode, applying a filter if provided.
- **Inputs**:
    - `modedata`: A pointer to a `window_customize_modedata` structure that holds the current state and data for the window customization mode.
    - `sort_crit`: An unused parameter intended for sorting criteria, represented as a pointer to a `mode_tree_sort_criteria` structure.
    - `tag`: An unused parameter intended for tagging, represented as a pointer to a `uint64_t`.
    - `filter`: A string used to filter the options and key bindings to be included in the customization mode.
- **Control Flow**:
    - The function begins by freeing any existing items in the `item_list` of the `modedata` and resets the list to NULL and size to 0.
    - It checks if the `fs` state in `modedata` is valid; if so, it copies it to a local `fs` variable, otherwise, it initializes `fs` from the pane in `modedata`.
    - A format tree `ft` is created from the `fs` state, and initial format variables `is_option` and `is_key` are set.
    - The function calls [`window_customize_build_options`](#window_customize_build_options) three times to build options for server, session, and window & pane, using different scopes and option trees, and passing the format tree and filter.
    - The format tree `ft` is freed and recreated from the `fs` state.
    - A loop iterates over key tables, calling [`window_customize_build_keys`](#window_customize_build_keys) for each non-empty key table, up to a maximum of 256 tables.
    - The format tree `ft` is freed at the end of the function.
- **Output**:
    - The function does not return a value; it modifies the `modedata` structure to populate it with the relevant options and key bindings.
- **Functions called**:
    - [`window_customize_free_item`](#window_customize_free_item)
    - [`cmd_find_valid_state`](cmd-find.c.driver.md#cmd_find_valid_state)
    - [`cmd_find_copy_state`](cmd-find.c.driver.md#cmd_find_copy_state)
    - [`cmd_find_from_pane`](cmd-find.c.driver.md#cmd_find_from_pane)
    - [`format_create_from_state`](format.c.driver.md#format_create_from_state)
    - [`window_customize_build_options`](#window_customize_build_options)
    - [`format_free`](format.c.driver.md#format_free)
    - [`key_bindings_first_table`](key-bindings.c.driver.md#key_bindings_first_table)
    - [`window_customize_build_keys`](#window_customize_build_keys)
    - [`key_bindings_next_table`](key-bindings.c.driver.md#key_bindings_next_table)


---
### window_customize_draw_key<!-- {{#callable:window_customize_draw_key}} -->
The `window_customize_draw_key` function displays information about a specific key binding in a customizable window interface.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure, which is unused in this function.
    - `item`: A pointer to a `window_customize_itemdata` structure representing the key binding item to be drawn.
    - `ctx`: A pointer to a `screen_write_ctx` structure used for writing to the screen.
    - `sx`: An unsigned integer representing the width of the screen area available for drawing.
    - `sy`: An unsigned integer representing the height of the screen area available for drawing.
- **Control Flow**:
    - Check if the `item` is NULL or if the key binding cannot be retrieved using [`window_customize_get_key`](#window_customize_get_key); if so, return immediately.
    - Retrieve the note associated with the key binding; if it is NULL, use a default message.
    - Append a period to the note if it does not already end with one.
    - Write the note to the screen using `screen_write_text`; if unsuccessful, return.
    - Move the cursor down one line using [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove); if the cursor is out of bounds, return.
    - Write the key table name and repeat status to the screen; if unsuccessful, return.
    - Move the cursor down one line again; if the cursor is out of bounds, return.
    - Retrieve and write the command associated with the key binding to the screen; if unsuccessful, free the command string and return.
    - Retrieve the default key binding command, compare it with the current command, and write it to the screen if they differ; free the command strings afterwards.
- **Output**:
    - The function does not return a value; it writes information about a key binding to the screen.
- **Functions called**:
    - [`window_customize_get_key`](#window_customize_get_key)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`cmd_list_print`](cmd.c.driver.md#cmd_list_print)
    - [`key_bindings_get_default`](key-bindings.c.driver.md#key_bindings_get_default)


---
### window_customize_draw_option<!-- {{#callable:window_customize_draw_option}} -->
The `window_customize_draw_option` function renders the details of a customizable option in a window, including its description, scope, value, and other attributes, onto a screen context.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure containing mode-specific data.
    - `item`: A pointer to a `window_customize_itemdata` structure representing the specific option item to be drawn.
    - `ctx`: A pointer to a `screen_write_ctx` structure used for writing to the screen.
    - `sx`: An unsigned integer representing the width of the screen area available for drawing.
    - `sy`: An unsigned integer representing the height of the screen area available for drawing.
- **Control Flow**:
    - Check if the item is valid using [`window_customize_check_item`](#window_customize_check_item); if not, return immediately.
    - Retrieve the option entry `o` using [`options_get`](options.c.driver.md#options_get) and the item's name; if not found, return.
    - Retrieve the options table entry `oe` for the option `o`.
    - If `oe` has a unit, set `space` and `unit` accordingly.
    - Create a format tree `ft` from the current state.
    - Write the option's description to the screen; if it fails, jump to cleanup.
    - Move the cursor to the next line and check if the screen has space; if not, jump to cleanup.
    - Determine the scope of the option and write it to the screen; if it fails, jump to cleanup.
    - If the option is an array, write the array information to the screen; if it fails, jump to cleanup.
    - Retrieve the option's value as a string and write it to the screen; if it fails, jump to cleanup.
    - If the option is a string, expand its value and write the expanded value if different; if it fails, jump to cleanup.
    - If the option is a choice, list available choices and write them to the screen; if it fails, jump to cleanup.
    - If the option is a color, write a color example to the screen; if it fails, jump to cleanup.
    - If the option is a style, apply the style and write an example to the screen; if it fails, jump to cleanup.
    - If a default value exists and is different from the current value, write it to the screen; if it fails, jump to cleanup.
    - Check for parent options and write their values if different from the current option; if it fails, jump to cleanup.
    - Cleanup: free allocated memory and format tree.
- **Output**:
    - The function does not return a value; it writes information to the screen context provided.
- **Functions called**:
    - [`window_customize_check_item`](#window_customize_check_item)
    - [`options_get`](options.c.driver.md#options_get)
    - [`options_table_entry`](tmux.h.driver.md#options_table_entry)
    - [`format_create_from_state`](format.c.driver.md#format_create_from_state)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`options_to_string`](options.c.driver.md#options_to_string)
    - [`options_default_to_string`](options.c.driver.md#options_default_to_string)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`style_apply`](style.c.driver.md#style_apply)
    - [`options_get_parent`](options.c.driver.md#options_get_parent)
    - [`options_owner`](options.c.driver.md#options_owner)
    - [`options_get_only`](options.c.driver.md#options_get_only)
    - [`format_free`](format.c.driver.md#format_free)


---
### window_customize_draw<!-- {{#callable:window_customize_draw}} -->
The `window_customize_draw` function determines whether to draw a key or an option in a window customization context based on the item's scope.
- **Inputs**:
    - `modedata`: A pointer to the window customization mode data, which contains information about the current state of the window customization.
    - `itemdata`: A pointer to the window customization item data, which represents the specific item to be drawn.
    - `ctx`: A pointer to the screen write context, which is used to draw on the screen.
    - `sx`: The width of the screen area available for drawing.
    - `sy`: The height of the screen area available for drawing.
- **Control Flow**:
    - Check if the item data is NULL; if so, return immediately without drawing.
    - Determine the scope of the item: if it is a key, call [`window_customize_draw_key`](#window_customize_draw_key); otherwise, call [`window_customize_draw_option`](#window_customize_draw_option).
- **Output**:
    - The function does not return a value; it performs drawing operations on the screen based on the item's scope.
- **Functions called**:
    - [`window_customize_draw_key`](#window_customize_draw_key)
    - [`window_customize_draw_option`](#window_customize_draw_option)


---
### window_customize_menu<!-- {{#callable:window_customize_menu}} -->
The `window_customize_menu` function processes a key event for a window customization mode in a client, ensuring the mode data is valid and then delegating the key handling to the [`window_customize_key`](#window_customize_key) function.
- **Inputs**:
    - `modedata`: A pointer to the mode data, specifically a `window_customize_modedata` structure, which contains information about the window pane and customization state.
    - `c`: A pointer to the `client` structure representing the client that triggered the key event.
    - `key`: A `key_code` representing the key that was pressed by the user.
- **Control Flow**:
    - Retrieve the `window_customize_modedata` structure from the `modedata` pointer.
    - Access the `window_pane` associated with the mode data.
    - Get the first `window_mode_entry` from the pane's mode queue using `TAILQ_FIRST`.
    - Check if the mode entry is `NULL` or if its data does not match the provided `modedata`; if so, return immediately.
    - Call [`window_customize_key`](#window_customize_key) with the mode entry, client, and key to handle the key event.
- **Output**:
    - The function does not return any value; it performs actions based on the key event and the current mode state.
- **Functions called**:
    - [`window_customize_key`](#window_customize_key)


---
### window_customize_height<!-- {{#callable:window_customize_height}} -->
The `window_customize_height` function returns a fixed height value of 12.
- **Inputs**:
    - `modedata`: This is an unused parameter, typically intended to hold mode-specific data.
    - `height`: This is an unused parameter, typically intended to represent the current height.
- **Control Flow**:
    - The function takes two parameters, both marked as unused, indicating they are not utilized within the function body.
    - The function directly returns the integer value 12 without any conditional logic or computation.
- **Output**:
    - The function returns an unsigned integer value of 12.


---
### window_customize_init<!-- {{#callable:window_customize_init}} -->
The `window_customize_init` function initializes a customization mode for a window pane, setting up a mode tree for managing options and key bindings.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the mode entry for the window pane.
    - `fs`: A pointer to a `cmd_find_state` structure used to find the current state of the command.
    - `args`: A pointer to an `args` structure containing command-line arguments, which may include format and prompt flags.
- **Control Flow**:
    - Allocate memory for a `window_customize_modedata` structure and assign it to `wme->data`.
    - Initialize the `window_customize_modedata` structure with the window pane and command find state.
    - Determine the format string to use based on the presence of the 'F' argument in `args`, defaulting to `WINDOW_CUSTOMIZE_DEFAULT_FORMAT` if not provided.
    - Set the `prompt_flags` to `PROMPT_ACCEPT` if the 'y' argument is present in `args`.
    - Start a mode tree using [`mode_tree_start`](mode-tree.c.driver.md#mode_tree_start), passing various callbacks and parameters, and store the result in `data->data`.
    - Apply zoom settings to the mode tree using [`mode_tree_zoom`](mode-tree.c.driver.md#mode_tree_zoom).
    - Build and draw the mode tree using [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build) and [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw).
    - Return the screen associated with the mode tree.
- **Output**:
    - Returns a pointer to a `screen` structure that represents the initialized mode tree screen.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`mode_tree_start`](mode-tree.c.driver.md#mode_tree_start)
    - [`mode_tree_zoom`](mode-tree.c.driver.md#mode_tree_zoom)
    - [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build)
    - [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw)


---
### window_customize_destroy<!-- {{#callable:window_customize_destroy}} -->
The `window_customize_destroy` function deallocates memory associated with a `window_customize_modedata` structure when its reference count reaches zero.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure that contains the data to be destroyed.
- **Control Flow**:
    - Decrement the `references` count of the `data` structure.
    - If the `references` count is not zero, return immediately without doing anything further.
    - Iterate over each item in the `item_list` array, calling [`window_customize_free_item`](#window_customize_free_item) to free each item.
    - Free the `item_list` array itself.
    - Free the `format` string.
    - Free the `data` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided data structure.
- **Functions called**:
    - [`window_customize_free_item`](#window_customize_free_item)


---
### window_customize_free<!-- {{#callable:window_customize_free}} -->
The `window_customize_free` function releases resources associated with a window customization mode entry by marking it as dead, freeing its mode tree data, and destroying the mode data structure.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` which contains the mode data to be freed.
- **Control Flow**:
    - Retrieve the `window_customize_modedata` from the `wme` structure.
    - Check if the `data` is `NULL`; if so, return immediately as there is nothing to free.
    - Set the `dead` flag of the `data` to 1, indicating that the mode data is no longer active.
    - Call [`mode_tree_free`](mode-tree.c.driver.md#mode_tree_free) to release the resources associated with the mode tree data within `data`.
    - Call [`window_customize_destroy`](#window_customize_destroy) to clean up and free the `window_customize_modedata` structure.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided mode entry.
- **Functions called**:
    - [`mode_tree_free`](mode-tree.c.driver.md#mode_tree_free)
    - [`window_customize_destroy`](#window_customize_destroy)


---
### window_customize_resize<!-- {{#callable:window_customize_resize}} -->
The `window_customize_resize` function adjusts the size of a mode tree associated with a window mode entry to specified dimensions.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry`, which contains data about the current window mode.
    - `sx`: An unsigned integer representing the new width to resize the mode tree to.
    - `sy`: An unsigned integer representing the new height to resize the mode tree to.
- **Control Flow**:
    - Retrieve the `window_customize_modedata` structure from the `data` field of the `window_mode_entry` structure pointed to by `wme`.
    - Call the [`mode_tree_resize`](mode-tree.c.driver.md#mode_tree_resize) function with the `data` field of the `window_customize_modedata` structure, and the new dimensions `sx` and `sy`.
- **Output**:
    - This function does not return a value; it performs an operation to resize the mode tree.
- **Functions called**:
    - [`mode_tree_resize`](mode-tree.c.driver.md#mode_tree_resize)


---
### window_customize_free_callback<!-- {{#callable:window_customize_free_callback}} -->
The `window_customize_free_callback` function is responsible for freeing resources associated with a window customization mode data structure.
- **Inputs**:
    - `modedata`: A pointer to the window customization mode data structure that needs to be destroyed.
- **Control Flow**:
    - The function calls [`window_customize_destroy`](#window_customize_destroy) with the `modedata` pointer to free the associated resources.
- **Output**:
    - This function does not return any value.
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
    - The function does not return any value; it performs memory cleanup operations.
- **Functions called**:
    - [`window_customize_free_item`](#window_customize_free_item)
    - [`window_customize_destroy`](#window_customize_destroy)


---
### window_customize_set_option_callback<!-- {{#callable:window_customize_set_option_callback}} -->
The `window_customize_set_option_callback` function updates a specified option for a window customization item, handling both array and non-array options, and manages error reporting if the update fails.
- **Inputs**:
    - `c`: A pointer to a `client` structure, representing the client that initiated the option change.
    - `itemdata`: A pointer to a `window_customize_itemdata` structure, which contains information about the specific option item being customized.
    - `s`: A constant character pointer representing the new value to set for the option.
    - `done`: An integer flag indicating whether the operation is complete, though it is unused in this function.
- **Control Flow**:
    - Check if the input string `s` is NULL, empty, or if the data is marked as dead; if so, return 0 immediately.
    - Verify the validity of the `item` by checking if it is NULL or fails the [`window_customize_check_item`](#window_customize_check_item) function; return 0 if invalid.
    - Retrieve the options entry `o` using [`options_get`](options.c.driver.md#options_get) with the item's options owner and name; return 0 if not found.
    - Determine if the option is an array by checking the flags of the options table entry `oe`; handle array and non-array options differently.
    - For array options, find the next available index if `idx` is -1, then attempt to set the array option using [`options_array_set`](options.c.driver.md#options_array_set); handle failure by jumping to the `fail` label.
    - For non-array options, attempt to set the option using [`options_from_string`](options.c.driver.md#options_from_string); handle failure by jumping to the `fail` label.
    - If successful, push changes to the options, rebuild and redraw the mode tree, and set the pane redraw flag.
    - In case of failure, capitalize the first character of the error message, display it using `status_message_set`, and free the error message.
- **Output**:
    - The function returns an integer, always 0, indicating the completion of the operation, regardless of success or failure.
- **Functions called**:
    - [`window_customize_check_item`](#window_customize_check_item)
    - [`options_get`](options.c.driver.md#options_get)
    - [`options_table_entry`](tmux.h.driver.md#options_table_entry)
    - [`options_array_get`](options.c.driver.md#options_array_get)
    - [`options_array_set`](options.c.driver.md#options_array_set)
    - [`options_from_string`](options.c.driver.md#options_from_string)
    - [`options_push_changes`](options.c.driver.md#options_push_changes)
    - [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build)
    - [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw)


---
### window_customize_set_option<!-- {{#callable:window_customize_set_option}} -->
The `window_customize_set_option` function sets or modifies an option for a window customization item, handling different scopes and types of options.
- **Inputs**:
    - `c`: A pointer to the client structure, representing the client for which the option is being set.
    - `data`: A pointer to the window_customize_modedata structure, which contains the mode data for the window customization.
    - `item`: A pointer to the window_customize_itemdata structure, representing the specific item whose option is being set.
    - `global`: An integer flag indicating whether the option should be set globally.
    - `pane`: An integer flag indicating whether the option should be set for a specific pane.
- **Control Flow**:
    - Check if the item is NULL or if the item is not valid using [`window_customize_check_item`](#window_customize_check_item); return if invalid.
    - Retrieve the options entry `o` for the item using [`options_get`](options.c.driver.md#options_get); return if not found.
    - Determine the options table entry `oe` and adjust the `pane` flag based on the entry's scope.
    - Determine the scope and options object `oo` based on the `global` flag and the item's current scope.
    - If the option is a flag, toggle its value using [`options_set_number`](options.c.driver.md#options_set_number).
    - If the option is a choice, cycle through the available choices using [`options_set_number`](options.c.driver.md#options_set_number).
    - For other types, prepare a prompt and value string for user input, and set up a new item for the prompt callback.
    - Increment the data's reference count and set a status prompt using [`status_prompt_set`](status.c.driver.md#status_prompt_set).
- **Output**:
    - The function does not return a value; it modifies the state of the options and may set a status prompt for user interaction.
- **Functions called**:
    - [`window_customize_check_item`](#window_customize_check_item)
    - [`options_get`](options.c.driver.md#options_get)
    - [`options_table_entry`](tmux.h.driver.md#options_table_entry)
    - [`window_customize_get_tree`](#window_customize_get_tree)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`options_set_number`](options.c.driver.md#options_set_number)
    - [`window_customize_scope_text`](#window_customize_scope_text)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`options_to_string`](options.c.driver.md#options_to_string)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`status_prompt_set`](status.c.driver.md#status_prompt_set)


---
### window_customize_unset_option<!-- {{#callable:window_customize_unset_option}} -->
The function `window_customize_unset_option` removes or resets an option entry for a given item in a window customization mode.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure, which contains the state and data for the window customization mode.
    - `item`: A pointer to a `window_customize_itemdata` structure, representing the specific item (option) to be unset or reset.
- **Control Flow**:
    - Check if the `item` is NULL or if the item is not valid by calling [`window_customize_check_item`](#window_customize_check_item); if either is true, return immediately.
    - Retrieve the options entry `o` using [`options_get`](options.c.driver.md#options_get) with the item's options owner and name.
    - If the options entry `o` is NULL, return immediately.
    - If the item's index is not -1 and the item is the current item in the mode tree, move up in the mode tree using [`mode_tree_up`](mode-tree.c.driver.md#mode_tree_up).
    - Call [`options_remove_or_default`](options.c.driver.md#options_remove_or_default) to remove or reset the option entry `o` at the specified index.
- **Output**:
    - The function does not return a value; it performs operations on the options entry associated with the given item.
- **Functions called**:
    - [`window_customize_check_item`](#window_customize_check_item)
    - [`options_get`](options.c.driver.md#options_get)
    - [`mode_tree_get_current`](mode-tree.c.driver.md#mode_tree_get_current)
    - [`mode_tree_up`](mode-tree.c.driver.md#mode_tree_up)
    - [`options_remove_or_default`](options.c.driver.md#options_remove_or_default)


---
### window_customize_reset_option<!-- {{#callable:window_customize_reset_option}} -->
The `window_customize_reset_option` function resets a specific option to its default value in a hierarchical options structure, if applicable.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure, which contains the context and state for the customization mode.
    - `item`: A pointer to a `window_customize_itemdata` structure, representing the specific option item to be reset.
- **Control Flow**:
    - Check if the `item` is NULL or if it fails the [`window_customize_check_item`](#window_customize_check_item) check; if so, return immediately.
    - Check if the `item` has a non-negative index (`item->idx != -1`); if so, return immediately as it indicates an array element which should not be reset here.
    - Initialize `oo` with the options object from the `item`.
    - Enter a loop that continues as long as `oo` is not NULL.
    - Within the loop, retrieve the options entry `o` for the current `oo` and `item->name`.
    - If `o` is not NULL, call [`options_remove_or_default`](options.c.driver.md#options_remove_or_default) to reset the option to its default value.
    - Update `oo` to its parent options object using [`options_get_parent`](options.c.driver.md#options_get_parent).
- **Output**:
    - The function does not return a value; it performs operations directly on the options data structures to reset the specified option.
- **Functions called**:
    - [`window_customize_check_item`](#window_customize_check_item)
    - [`options_get_only`](options.c.driver.md#options_get_only)
    - [`options_remove_or_default`](options.c.driver.md#options_remove_or_default)
    - [`options_get_parent`](options.c.driver.md#options_get_parent)


---
### window_customize_set_command_callback<!-- {{#callable:window_customize_set_command_callback}} -->
The `window_customize_set_command_callback` function processes a command string for a key binding, updating the command list if the parsing is successful, or displaying an error message if parsing fails.
- **Inputs**:
    - `c`: A pointer to a `client` structure, representing the client context in which the command is being processed.
    - `itemdata`: A pointer to a `window_customize_itemdata` structure, which contains data about the item being customized, including its associated mode data.
    - `s`: A constant character pointer representing the command string to be parsed and set for the key binding.
    - `done`: An integer flag indicating whether the operation is complete, but it is unused in this function.
- **Control Flow**:
    - Check if the command string `s` is NULL, empty, or if the mode data is marked as dead; if so, return 0 immediately.
    - Retrieve the `window_customize_itemdata` and `window_customize_modedata` from `itemdata`.
    - Check if `itemdata` is NULL or if the key binding cannot be retrieved; if so, return 0.
    - Parse the command string `s` using `cmd_parse_from_string`.
    - If parsing results in an error, convert the first character of the error message to uppercase, display the error message using `status_message_set`, free the error string, and return 0.
    - If parsing is successful, free the existing command list in the key binding and set it to the newly parsed command list.
    - Rebuild and redraw the mode tree using [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build) and [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw).
    - Set the `PANE_REDRAW` flag on the window pane to indicate that it needs to be redrawn.
- **Output**:
    - The function returns an integer value of 0, indicating the completion of the callback operation, regardless of success or failure.
- **Functions called**:
    - [`window_customize_get_key`](#window_customize_get_key)
    - [`cmd_list_free`](cmd.c.driver.md#cmd_list_free)
    - [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build)
    - [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw)


---
### window_customize_set_note_callback<!-- {{#callable:window_customize_set_note_callback}} -->
The function `window_customize_set_note_callback` updates the note of a key binding in a window customization mode if the input string is valid and the data is not marked as dead.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `itemdata`: A pointer to `window_customize_itemdata` structure, which contains data about the item being customized.
    - `s`: A constant character pointer representing the new note to be set for the key binding.
    - `done`: An integer indicating if the operation is complete, which is unused in this function.
- **Control Flow**:
    - Check if the input string `s` is NULL, empty, or if the data is marked as dead; if so, return 0.
    - Retrieve the `window_customize_itemdata` and `window_customize_modedata` from `itemdata`.
    - Check if the item is NULL or if the key binding cannot be retrieved using [`window_customize_get_key`](#window_customize_get_key); if so, return 0.
    - Free the existing note of the key binding and duplicate the new note string `s` using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - Rebuild and redraw the mode tree using [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build) and [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw).
    - Set the `PANE_REDRAW` flag on the window pane to indicate it needs to be redrawn.
    - Return 0.
- **Output**:
    - The function returns an integer value 0, indicating the operation was completed without errors.
- **Functions called**:
    - [`window_customize_get_key`](#window_customize_get_key)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build)
    - [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw)


---
### window_customize_set_key<!-- {{#callable:window_customize_set_key}} -->
The `window_customize_set_key` function customizes key bindings for a window by setting or modifying key commands, notes, or repeat flags based on the current mode tree selection.
- **Inputs**:
    - `c`: A pointer to the client structure, representing the client for which the key customization is being set.
    - `data`: A pointer to the `window_customize_modedata` structure, which contains the mode data for the window customization process.
    - `item`: A pointer to the `window_customize_itemdata` structure, representing the specific item (key) being customized.
- **Control Flow**:
    - Check if the `item` is NULL or if the key binding cannot be retrieved using [`window_customize_get_key`](#window_customize_get_key); if so, return immediately.
    - Retrieve the current name from the mode tree using [`mode_tree_get_current_name`](mode-tree.c.driver.md#mode_tree_get_current_name).
    - If the current name is 'Repeat', toggle the `KEY_BINDING_REPEAT` flag for the key binding.
    - If the current name is 'Command', prepare a prompt and value for the command associated with the key, allocate a new item, and set a status prompt for command customization.
    - If the current name is 'Note', prepare a prompt for the note associated with the key, allocate a new item, and set a status prompt for note customization.
- **Output**:
    - The function does not return a value; it modifies the key binding settings and may set a status prompt for further user input.
- **Functions called**:
    - [`window_customize_get_key`](#window_customize_get_key)
    - [`mode_tree_get_current_name`](mode-tree.c.driver.md#mode_tree_get_current_name)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)
    - [`cmd_list_print`](cmd.c.driver.md#cmd_list_print)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`status_prompt_set`](status.c.driver.md#status_prompt_set)


---
### window_customize_unset_key<!-- {{#callable:window_customize_unset_key}} -->
The `window_customize_unset_key` function removes a key binding from a key table if the specified item is valid and currently selected.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure, which contains the mode data for the window customization.
    - `item`: A pointer to a `window_customize_itemdata` structure, which represents the item data for the key to be unset.
- **Control Flow**:
    - Check if the `item` is NULL or if [`window_customize_get_key`](#window_customize_get_key) returns false; if so, return immediately.
    - If the `item` is the current item in the mode tree, collapse the current item and move up in the mode tree.
    - Remove the key binding from the key table using [`key_bindings_remove`](key-bindings.c.driver.md#key_bindings_remove).
- **Output**:
    - This function does not return a value; it performs operations to modify the key bindings.
- **Functions called**:
    - [`window_customize_get_key`](#window_customize_get_key)
    - [`mode_tree_get_current`](mode-tree.c.driver.md#mode_tree_get_current)
    - [`mode_tree_collapse_current`](mode-tree.c.driver.md#mode_tree_collapse_current)
    - [`mode_tree_up`](mode-tree.c.driver.md#mode_tree_up)
    - [`key_bindings_remove`](key-bindings.c.driver.md#key_bindings_remove)


---
### window_customize_reset_key<!-- {{#callable:window_customize_reset_key}} -->
The `window_customize_reset_key` function resets a key binding to its default state in a specified key table if it differs from the default.
- **Inputs**:
    - `data`: A pointer to a `window_customize_modedata` structure, which contains the mode data for the window customization.
    - `item`: A pointer to a `window_customize_itemdata` structure, which represents the item data for the key to be reset.
- **Control Flow**:
    - Check if `item` is NULL or if [`window_customize_get_key`](#window_customize_get_key) fails to retrieve the key table and binding; if so, return immediately.
    - Retrieve the default key binding `dd` for the specified key in the key table `kt`.
    - If the current binding `bd` matches the default binding `dd`, return without making changes.
    - If there is no default binding `dd` and the item is the current item in the mode tree, collapse the current mode tree item and move up in the mode tree.
    - Reset the key binding in the key table `kt` to its default state using [`key_bindings_reset`](key-bindings.c.driver.md#key_bindings_reset).
- **Output**:
    - The function does not return a value; it performs operations to reset a key binding to its default state.
- **Functions called**:
    - [`window_customize_get_key`](#window_customize_get_key)
    - [`key_bindings_get_default`](key-bindings.c.driver.md#key_bindings_get_default)
    - [`mode_tree_get_current`](mode-tree.c.driver.md#mode_tree_get_current)
    - [`mode_tree_collapse_current`](mode-tree.c.driver.md#mode_tree_collapse_current)
    - [`mode_tree_up`](mode-tree.c.driver.md#mode_tree_up)
    - [`key_bindings_reset`](key-bindings.c.driver.md#key_bindings_reset)


---
### window_customize_change_each<!-- {{#callable:window_customize_change_each}} -->
The `window_customize_change_each` function modifies a window customization item based on its scope and the specified change type, either unsetting or resetting the item.
- **Inputs**:
    - `modedata`: A pointer to `window_customize_modedata`, which contains the mode data for window customization.
    - `itemdata`: A pointer to `window_customize_itemdata`, which represents the specific item to be customized.
    - `c`: An unused parameter representing a client structure.
    - `key`: An unused parameter representing a key code.
- **Control Flow**:
    - The function begins by casting `modedata` and `itemdata` to their respective types, `window_customize_modedata` and `window_customize_itemdata`.
    - A switch statement is used to determine the action based on `data->change`.
    - If `data->change` is `WINDOW_CUSTOMIZE_UNSET`, the function checks the `item->scope`. If the scope is `WINDOW_CUSTOMIZE_KEY`, it calls [`window_customize_unset_key`](#window_customize_unset_key); otherwise, it calls [`window_customize_unset_option`](#window_customize_unset_option).
    - If `data->change` is `WINDOW_CUSTOMIZE_RESET`, the function again checks the `item->scope`. If the scope is `WINDOW_CUSTOMIZE_KEY`, it calls [`window_customize_reset_key`](#window_customize_reset_key); otherwise, it calls [`window_customize_reset_option`](#window_customize_reset_option).
    - After handling the change, if the `item->scope` is not `WINDOW_CUSTOMIZE_KEY`, it calls [`options_push_changes`](options.c.driver.md#options_push_changes) with `item->name` to apply the changes.
- **Output**:
    - The function does not return a value; it performs actions on the provided data structures to modify window customization items.
- **Functions called**:
    - [`window_customize_unset_key`](#window_customize_unset_key)
    - [`window_customize_unset_option`](#window_customize_unset_option)
    - [`window_customize_reset_key`](#window_customize_reset_key)
    - [`window_customize_reset_option`](#window_customize_reset_option)
    - [`options_push_changes`](options.c.driver.md#options_push_changes)


---
### window_customize_change_current_callback<!-- {{#callable:window_customize_change_current_callback}} -->
The function `window_customize_change_current_callback` processes a confirmation input to either unset or reset a customization item in a window mode, based on the current mode data and item scope.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `modedata`: A pointer to `window_customize_modedata` structure, which contains the mode data for the current window customization.
    - `s`: A constant character pointer representing the input string, which is expected to be a confirmation input ('y' for yes).
    - `done`: An integer flag indicating if the operation is done, which is unused in this function.
- **Control Flow**:
    - Check if the input string `s` is NULL, empty, or if the mode data is marked as dead; if any of these conditions are true, return 0.
    - Convert the first character of `s` to lowercase and check if it is 'y' and the string length is 1; if not, return 0.
    - Retrieve the current item from the mode tree using [`mode_tree_get_current`](mode-tree.c.driver.md#mode_tree_get_current).
    - Switch on the `change` field of the mode data to determine whether to unset or reset the current item.
    - If the item's scope is not `WINDOW_CUSTOMIZE_KEY`, call [`options_push_changes`](options.c.driver.md#options_push_changes) with the item's name to apply changes.
    - Rebuild and redraw the mode tree using [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build) and [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw).
    - Set the `PANE_REDRAW` flag on the window pane to indicate that it needs to be redrawn.
- **Output**:
    - The function returns 0, indicating the operation was processed without errors.
- **Functions called**:
    - [`mode_tree_get_current`](mode-tree.c.driver.md#mode_tree_get_current)
    - [`window_customize_unset_key`](#window_customize_unset_key)
    - [`window_customize_unset_option`](#window_customize_unset_option)
    - [`window_customize_reset_key`](#window_customize_reset_key)
    - [`window_customize_reset_option`](#window_customize_reset_option)
    - [`options_push_changes`](options.c.driver.md#options_push_changes)
    - [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build)
    - [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw)


---
### window_customize_change_tagged_callback<!-- {{#callable:window_customize_change_tagged_callback}} -->
The function `window_customize_change_tagged_callback` processes a confirmation input to apply changes to all tagged items in a mode tree, rebuilding and redrawing the tree if confirmed.
- **Inputs**:
    - `c`: A pointer to a `client` structure, representing the client that triggered the callback.
    - `modedata`: A pointer to `window_customize_modedata`, which contains the mode data for the window customization.
    - `s`: A constant character pointer representing the input string, typically a confirmation response.
    - `done`: An integer flag indicating if the operation is complete, but it is unused in this function.
- **Control Flow**:
    - Check if the input string `s` is NULL, empty, or if the `data->dead` flag is set; if any of these conditions are true, return 0 immediately.
    - Convert the first character of `s` to lowercase and check if it is 'y' and if `s` has no other characters; if not, return 0.
    - Call [`mode_tree_each_tagged`](mode-tree.c.driver.md#mode_tree_each_tagged) to apply the `window_customize_change_each` function to each tagged item in the mode tree, passing the client `c` and other parameters.
    - Rebuild the mode tree using [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build).
    - Redraw the mode tree using [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw).
    - Set the `PANE_REDRAW` flag on `data->wp` to indicate that the pane needs to be redrawn.
    - Return 0.
- **Output**:
    - The function returns an integer value, always 0, indicating the operation's completion status.
- **Functions called**:
    - [`mode_tree_each_tagged`](mode-tree.c.driver.md#mode_tree_each_tagged)
    - [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build)
    - [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw)


---
### window_customize_key<!-- {{#callable:window_customize_key}} -->
The `window_customize_key` function handles key events for customizing window options and key bindings in a mode tree interface.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the current window mode entry.
    - `c`: A pointer to a `client` structure representing the client interacting with the window.
    - `s`: An unused pointer to a `session` structure.
    - `wl`: An unused pointer to a `winlink` structure.
    - `key`: A `key_code` representing the key event that triggered the function.
    - `m`: A pointer to a `mouse_event` structure representing any mouse event associated with the key event.
- **Control Flow**:
    - Retrieve the current item from the mode tree using [`mode_tree_get_current`](mode-tree.c.driver.md#mode_tree_get_current) and store it in `item`.
    - Process the key event using [`mode_tree_key`](mode-tree.c.driver.md#mode_tree_key), updating the `finished` status and potentially changing the current item.
    - If the key is '\r' or 's', set the key or option based on the item's scope and rebuild the mode tree.
    - If the key is 'w', set the option without pane scope and rebuild the mode tree.
    - If the key is 'S' or 'W', set the option with global scope and rebuild the mode tree.
    - If the key is 'd', prompt the user to reset the current item to default if applicable.
    - If the key is 'D', prompt the user to reset all tagged items to default if any are tagged.
    - If the key is 'u', prompt the user to unset the current item if applicable.
    - If the key is 'U', prompt the user to unset all tagged items if any are tagged.
    - If the key is 'H', toggle the visibility of global options and rebuild the mode tree.
    - If the `finished` flag is set, reset the window pane mode; otherwise, redraw the mode tree and set the pane redraw flag.
- **Output**:
    - The function does not return a value; it modifies the state of the mode tree and window pane based on the key event.
- **Functions called**:
    - [`mode_tree_get_current`](mode-tree.c.driver.md#mode_tree_get_current)
    - [`mode_tree_key`](mode-tree.c.driver.md#mode_tree_key)
    - [`window_customize_set_key`](#window_customize_set_key)
    - [`window_customize_set_option`](#window_customize_set_option)
    - [`options_push_changes`](options.c.driver.md#options_push_changes)
    - [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`status_prompt_set`](status.c.driver.md#status_prompt_set)
    - [`mode_tree_count_tagged`](mode-tree.c.driver.md#mode_tree_count_tagged)
    - [`window_pane_reset_mode`](window.c.driver.md#window_pane_reset_mode)
    - [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw)


