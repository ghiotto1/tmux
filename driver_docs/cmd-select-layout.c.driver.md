# Purpose
This C source code file is part of the tmux terminal multiplexer project, specifically handling the functionality related to window layout management. The file defines commands for selecting, cycling to the next, and reverting to the previous window layout within a tmux session. The primary technical component is the [`cmd_select_layout_exec`](#cmd_select_layout_exec) function, which executes the logic for these layout commands. It interacts with the tmux server to adjust window layouts, either by selecting a specific layout by name, cycling through available layouts, or reverting to a previously used layout. The function also handles special cases such as spreading out panes or using the last known layout.

The file defines three command entries: `select-layout`, `next-layout`, and `previous-layout`, each associated with specific command-line arguments and usage patterns. These commands are registered with the tmux command system and are executed through the [`cmd_select_layout_exec`](#cmd_select_layout_exec) function. The code is structured to integrate with the broader tmux command framework, utilizing structures and functions from the tmux codebase to manipulate window and pane layouts. This file does not define a standalone executable but rather contributes to the tmux application's functionality by providing a modular component for layout management.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `tmux.h`


# Global Variables

---
### cmd_select_layout_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_select_layout_entry` is a constant structure of type `cmd_entry` that defines the command 'select-layout' and its alias 'selectl'. It specifies the arguments, usage, target, flags, and execution function for the command, which is used to switch a window to a selected layout in the tmux application.
- **Use**: This variable is used to register and define the behavior of the 'select-layout' command within the tmux application, allowing users to change the layout of a window pane.


---
### cmd_next_layout_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_next_layout_entry` is a constant structure of type `cmd_entry` that defines the command for switching to the next window layout in the tmux application. It specifies the command name as "next-layout" with an alias "nextl", and includes argument specifications, usage instructions, target specifications, and execution flags. The command is executed using the `cmd_select_layout_exec` function.
- **Use**: This variable is used to define and register the 'next-layout' command in tmux, allowing users to switch to the next available window layout.


---
### cmd_previous_layout_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_previous_layout_entry` is a constant structure of type `cmd_entry` that defines the command for switching to the previous window layout in the tmux application. It specifies the command name as "previous-layout" with an alias "prevl", and includes argument specifications, usage instructions, target specifications, and execution flags. The command is executed using the `cmd_select_layout_exec` function.
- **Use**: This variable is used to define and register the command for selecting the previous window layout in the tmux application.


# Functions

---
### cmd_select_layout_exec<!-- {{#callable:cmd_select_layout_exec}} -->
The `cmd_select_layout_exec` function changes the layout of a window in a terminal multiplexer based on specified arguments or commands.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the command arguments and target window and pane from the command queue item.
    - Unzoom the target window if it is zoomed.
    - Determine if the command is for the next or previous layout based on the command entry or arguments.
    - Store the current layout of the window and update it with the new layout dump.
    - If the command is for the next or previous layout, set the window to the next or previous layout and proceed to the 'changed' label.
    - If the 'E' argument is present, spread out the layout of the target pane and proceed to the 'changed' label.
    - Determine the layout name from the arguments or use the old layout if the 'o' argument is present.
    - If the layout name is not specified and the 'o' argument is not present, use the last layout or look up the layout by name.
    - If a valid layout is found, select it and proceed to the 'changed' label.
    - If a layout name is specified, parse it and handle errors if parsing fails, otherwise proceed to the 'changed' label.
    - Free the old layout and return normal command execution status if no changes are made.
    - At the 'changed' label, free the old layout, recalculate sizes, redraw the window, notify of layout change, and return normal command execution status.
    - At the 'error' label, restore the old layout, free the new layout, and return an error command execution status.
- **Output**:
    - Returns `CMD_RETURN_NORMAL` if the layout change is successful or no change is needed, and `CMD_RETURN_ERROR` if an error occurs during layout parsing.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`server_unzoom_window`](server-fn.c.driver.md#server_unzoom_window)
    - [`cmd_get_entry`](cmd.c.driver.md#cmd_get_entry)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`layout_dump`](layout-custom.c.driver.md#layout_dump)
    - [`layout_set_next`](layout-set.c.driver.md#layout_set_next)
    - [`layout_set_previous`](layout-set.c.driver.md#layout_set_previous)
    - [`layout_spread_out`](layout.c.driver.md#layout_spread_out)
    - [`args_count`](arguments.c.driver.md#args_count)
    - [`args_string`](arguments.c.driver.md#args_string)
    - [`layout_set_lookup`](layout-set.c.driver.md#layout_set_lookup)
    - [`layout_set_select`](layout-set.c.driver.md#layout_set_select)
    - [`layout_parse`](layout-custom.c.driver.md#layout_parse)
    - [`recalculate_sizes`](resize.c.driver.md#recalculate_sizes)
    - [`server_redraw_window`](server-fn.c.driver.md#server_redraw_window)
    - [`notify_window`](notify.c.driver.md#notify_window)


