# Purpose
This C source code file is part of the `tmux` terminal multiplexer project and implements the functionality to swap two panes within a terminal window. The file defines a command, `swap-pane`, which is used to exchange the positions of two panes, either within the same window or between different windows. The command is registered with a set of options and arguments that allow users to specify the source and target panes, as well as additional flags to control the behavior of the swap operation, such as whether to keep the panes detached or to adjust the zoom state of the windows involved.

The core functionality is encapsulated in the [`cmd_swap_pane_exec`](#cmd_swap_pane_exec) function, which handles the logic for identifying the source and target panes, performing the swap, and updating the layout and state of the windows and panes. This includes managing the pane's layout cells, resizing panes, and updating active pane states. The code ensures that the visual representation of the windows is updated by redrawing them as necessary and notifies the system of layout changes. This file is a focused implementation of a specific feature within the broader `tmux` application, providing a clear and well-defined interface for swapping panes, which is a common operation for users managing multiple terminal sessions.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `tmux.h`


# Global Variables

---
### cmd_swap_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_swap_pane_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'swap-pane' command in the tmux application. It includes the command's name, alias, argument specifications, usage information, source and target pane specifications, flags, and the function to execute the command.
- **Use**: This variable is used to register and define the behavior of the 'swap-pane' command, allowing users to swap two panes within tmux.


# Functions

---
### cmd_swap_pane_exec<!-- {{#callable:cmd_swap_pane_exec}} -->
The `cmd_swap_pane_exec` function swaps two panes within or between windows in a terminal multiplexer, handling layout adjustments and redrawing as necessary.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the arguments, source, and target states from the command and command queue item.
    - Identify the source and destination windows and panes from the source and target states.
    - Check if the destination window needs to be zoomed and redraw it if necessary.
    - Determine the source pane based on the 'D' or 'U' flags, adjusting the source window and pane accordingly.
    - If the source and destination panes are the same, exit the function early.
    - Remove the source and destination panes from their respective clients.
    - Swap the panes in the window's pane list, adjusting the layout cells and window references.
    - Update the pane sizes and offsets to match the new window configurations.
    - Set the active pane based on the 'd' flag and whether the source and destination windows are the same.
    - If the source and destination windows differ, update the pane stack and redraw the source window.
    - Fix the layout of the destination window and redraw it.
    - Notify that the window layout has changed for both source and destination windows if they differ.
    - Pop the zoom state for the source and destination windows if necessary and redraw them.
- **Output**:
    - The function returns `CMD_RETURN_NORMAL`, indicating successful execution.
- **Functions called**:
    - [`window_pane_resize`](tmux.h.driver.md#window_pane_resize)


# Function Declarations (Public API)

---
### cmd_swap_pane_exec<!-- {{#callable_declaration:cmd_swap_pane_exec}} -->
Swap two panes within or between windows.
- **Description**: This function swaps two panes, either within the same window or between different windows, based on the source and target specified. It can be used to rearrange panes for better organization or workflow. The function supports options to swap with the next or previous pane in the window, and can also handle zoomed panes. It must be called with valid source and target pane identifiers, and the panes must exist within the specified windows. The function ensures that the layout and active pane status are updated accordingly, and it triggers necessary redraws and notifications.
- **Inputs**:
    - `self`: A pointer to the command structure representing the swap-pane command. Must not be null.
    - `item`: A pointer to the command queue item, which provides context for the source and target panes. Must not be null.
- **Output**: Returns CMD_RETURN_NORMAL to indicate successful execution. The function updates the layout and active pane status of the involved windows and panes.
- **See also**: [`cmd_swap_pane_exec`](#cmd_swap_pane_exec)  (Implementation)


