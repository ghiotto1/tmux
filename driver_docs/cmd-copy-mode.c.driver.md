# Purpose
This C source code file is part of the tmux terminal multiplexer project and is responsible for implementing the functionality to enter either "copy mode" or "clock mode" within a tmux session. The file defines two command entries, `cmd_copy_mode_entry` and `cmd_clock_mode_entry`, which correspond to the "copy-mode" and "clock-mode" commands, respectively. These commands allow users to interact with tmux panes in specific ways: "copy mode" enables users to scroll through and copy text from the terminal output, while "clock mode" displays a clock within a pane. The file includes the implementation of the [`cmd_copy_mode_exec`](#cmd_copy_mode_exec) function, which executes the appropriate actions based on the command and its arguments, such as resetting modes, setting the pane mode, and handling mouse events for interaction.

The code is structured to be integrated into the larger tmux application, with the command entries and execution logic designed to be invoked by the tmux command queue system. The [`cmd_copy_mode_exec`](#cmd_copy_mode_exec) function handles various command-line arguments to modify its behavior, such as scrolling up or down, starting a drag operation, or switching between source and target panes. The file does not define a standalone executable but rather contributes to the tmux command infrastructure, providing specific functionality related to pane interaction modes. The use of static functions and the absence of public APIs indicate that this code is intended for internal use within the tmux application.
# Imports and Dependencies

---
- `sys/types.h`
- `tmux.h`


# Global Variables

---
### cmd_copy_mode_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_copy_mode_entry` is a constant structure of type `cmd_entry` that defines the properties and behavior of the 'copy-mode' command in the application. It includes the command's name, arguments, usage, source and target pane specifications, flags, and the function to execute when the command is invoked.
- **Use**: This variable is used to register and define the behavior of the 'copy-mode' command, allowing users to enter copy mode in the application.


---
### cmd_clock_mode_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_clock_mode_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'clock-mode' command in the tmux application. It specifies the command's name, arguments, usage, target, flags, and the function to execute when the command is invoked. This structure is used to configure and handle the 'clock-mode' command, which is likely related to displaying a clock in a specific pane.
- **Use**: This variable is used to define and handle the 'clock-mode' command within the tmux application, specifying its behavior and execution.


# Functions

---
### cmd_copy_mode_exec<!-- {{#callable:cmd_copy_mode_exec}} -->
The `cmd_copy_mode_exec` function handles the execution of entering copy or clock mode in a tmux window pane based on provided command arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve command arguments, event, source, target, and client from the provided structures.
    - Check if the 'q' argument is present; if so, reset the mode of the target window pane and return.
    - If the 'M' argument is present, attempt to get the window pane from the mouse event; if unsuccessful or client session mismatch, return.
    - Check if the command entry is for clock mode; if so, set the window pane to clock mode and return.
    - Determine the source window pane based on the presence of the 's' argument.
    - Attempt to set the window pane to copy mode; if unsuccessful and 'M' is present, start a drag operation.
    - Handle additional arguments: 'u' for page up, 'd' for page down, and 'S' for scrolling with mouse position.
    - Return normal command execution status.
- **Output**:
    - The function returns an `enum cmd_retval` indicating the result of the command execution, typically `CMD_RETURN_NORMAL`.


# Function Declarations (Public API)

---
### cmd_copy_mode_exec<!-- {{#callable_declaration:cmd_copy_mode_exec}} -->
Enter copy or clock mode for a window pane.
- **Description**: This function is used to switch a window pane into either copy mode or clock mode based on the command entry and provided arguments. It should be called when a user wants to interact with the content of a pane in a different mode, such as copying text or viewing a clock. The function handles various flags that modify its behavior, such as quitting the mode, using mouse events, or scrolling. It requires valid command and command queue item structures, and the target pane must be specified. The function ensures that the mode is set correctly and handles any necessary interactions, such as mouse drags or page scrolling.
- **Inputs**:
    - `self`: A pointer to the cmd structure representing the command to be executed. Must not be null.
    - `item`: A pointer to the cmdq_item structure representing the command queue item. Must not be null.
- **Output**: Returns an enum cmd_retval indicating the result of the command execution, typically CMD_RETURN_NORMAL.
- **See also**: [`cmd_copy_mode_exec`](#cmd_copy_mode_exec)  (Implementation)


