# Purpose
This C source code file is part of the `tmux` terminal multiplexer project and implements the functionality to swap two panes within a terminal window. The file defines a command, `swap-pane`, which allows users to interchange the positions of two panes, either within the same window or across different windows. The command is registered with a set of options and arguments that dictate its behavior, such as whether to swap with the next or previous pane, or to perform the swap without changing the active pane. The core functionality is encapsulated in the [`cmd_swap_pane_exec`](#cmd_swap_pane_exec) function, which handles the logic for identifying the source and target panes, performing the swap, and updating the layout and state of the windows and panes involved.

The code is structured to integrate with the broader `tmux` command framework, as evidenced by the use of structures like `cmd_entry` and functions such as `cmd_get_args` and `cmdq_get_source`. It manipulates data structures that represent windows and panes, such as `window`, `window_pane`, and `layout_cell`, to achieve the pane swapping. The file does not define a standalone executable but rather a command that can be invoked within the `tmux` environment, contributing to the extensibility and modularity of the `tmux` system. The code also includes mechanisms to handle user interface updates and notifications, ensuring that the visual representation of the terminal reflects the changes made by the command.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `tmux.h`


# Global Variables

---
### cmd_swap_pane_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_swap_pane_entry` is a constant structure of type `cmd_entry` that defines the command for swapping two panes in the tmux terminal multiplexer. It includes the command name, alias, argument specifications, usage information, source and target pane specifications, flags, and the execution function pointer.
- **Use**: This variable is used to register and define the behavior of the 'swap-pane' command within the tmux application.


# Functions

---
### cmd_swap_pane_exec<!-- {{#callable:cmd_swap_pane_exec}} -->
The `cmd_swap_pane_exec` function swaps two panes within or between windows in a terminal multiplexer, handling layout adjustments and redrawing as necessary.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item, which contains source and target pane information.
- **Control Flow**:
    - Retrieve arguments and source/target pane information from the command and command queue item.
    - Determine the source and destination windows and panes based on the command arguments.
    - If the 'Z' argument is present, push zoom on the destination window and redraw it.
    - If the 'D' or 'U' argument is present, adjust the source pane to be the next or previous pane in the destination window, respectively.
    - If the source and destination panes are the same, exit the function early.
    - Remove the source and destination panes from their respective client lists.
    - Swap the panes in their respective window pane lists, adjusting the layout cells accordingly.
    - Update the window references and options for the swapped panes, marking them as changed.
    - Resize the panes to match their new positions and sizes.
    - If the 'd' argument is not present, set the active pane in each window appropriately.
    - If the source and destination windows are different, update the pane stacks and redraw the source window.
    - Fix the layout of the destination window and redraw it.
    - Notify that the window layout has changed for both windows if they are different.
    - Pop zoom from the source and destination windows if it was previously pushed, and redraw them.
- **Output**:
    - The function returns `CMD_RETURN_NORMAL`, indicating successful execution of the pane swap.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_source`](cmd-queue.c.driver.md#cmdq_get_source)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`window_push_zoom`](window.c.driver.md#window_push_zoom)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`server_redraw_window`](server-fn.c.driver.md#server_redraw_window)
    - [`server_client_remove_pane`](server-client.c.driver.md#server_client_remove_pane)
    - [`options_set_parent`](options.c.driver.md#options_set_parent)
    - [`window_pane_resize`](tmux.h.driver.md#window_pane_resize)
    - [`window_set_active_pane`](window.c.driver.md#window_set_active_pane)
    - [`window_pane_stack_remove`](window.c.driver.md#window_pane_stack_remove)
    - [`colour_palette_from_option`](colour.c.driver.md#colour_palette_from_option)
    - [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes)
    - [`notify_window`](notify.c.driver.md#notify_window)
    - [`window_pop_zoom`](window.c.driver.md#window_pop_zoom)


