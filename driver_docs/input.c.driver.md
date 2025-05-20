# Purpose
The provided C source code is part of the `tmux` terminal multiplexer, specifically handling the parsing and processing of terminal input sequences. This file is responsible for interpreting various control sequences, escape sequences, and other input data that a terminal might receive. It includes functionality for handling ANSI and DEC terminal sequences, such as cursor movements, screen clearing, and color settings. The code is structured around a state machine that transitions between different states based on the input received, allowing it to handle complex sequences efficiently.

Key components of this code include the definition of various states and transitions for the input parser, handling of specific sequences like OSC (Operating System Command) for setting terminal colors and clipboard contents, and DCS (Device Control String) for more complex terminal interactions. The code also includes mechanisms for handling UTF-8 input, managing terminal modes, and responding to terminal queries. This file is integral to `tmux`'s ability to emulate a terminal environment, providing a robust interface for interpreting and responding to terminal input in a way that supports advanced features like color management, hyperlink embedding, and clipboard integration.
# Imports and Dependencies

---
- `sys/types.h`
- `netinet/in.h`
- `ctype.h`
- `resolv.h`
- `stdlib.h`
- `string.h`
- `time.h`
- `tmux.h`


# Data Structures

---
### input_cell
- **Type**: `struct`
- **Members**:
    - `cell`: A nested `struct grid_cell` representing the grid cell associated with this input cell.
    - `set`: An integer flag indicating whether the input cell is set.
    - `g0set`: An integer flag indicating if the G0 character set is active (1 if ACS).
    - `g1set`: An integer flag indicating if the G1 character set is active (1 if ACS).
- **Description**: The `input_cell` structure is used within an input parser to represent a cell in a grid, specifically for handling terminal input sequences. It contains a `grid_cell` structure to hold the cell's data and attributes, and additional flags to manage character set states, such as whether the G0 or G1 character set is active, which is relevant for handling alternate character sets (ACS) in terminal emulation.


---
### input_param
- **Type**: `struct`
- **Members**:
    - `type`: An enum indicating the type of input parameter, which can be missing, a number, or a string.
    - `num`: An integer value used when the input parameter is a number.
    - `str`: A pointer to a character string used when the input parameter is a string.
- **Description**: The `input_param` structure is used to represent a parameter in an input parsing context, where the parameter can be of different types: missing, numeric, or string. The `type` member is an enumeration that specifies the kind of parameter, while the union allows the parameter to be stored as either an integer (`num`) or a string (`str`), depending on the type. This structure is useful in scenarios where input parameters need to be parsed and handled dynamically based on their type.


---
### input_ctx
- **Type**: `struct`
- **Members**:
    - `wp`: Pointer to the associated window pane.
    - `event`: Pointer to the buffer event for input handling.
    - `ctx`: Screen write context for managing screen updates.
    - `palette`: Pointer to the colour palette used for rendering.
    - `cell`: Current input cell being processed.
    - `old_cell`: Previous input cell state.
    - `old_cx`: Previous cursor x-coordinate.
    - `old_cy`: Previous cursor y-coordinate.
    - `old_mode`: Previous screen mode.
    - `interm_buf`: Buffer for intermediate characters in escape sequences.
    - `interm_len`: Length of the intermediate buffer.
    - `param_buf`: Buffer for parameters in escape sequences.
    - `param_len`: Length of the parameter buffer.
    - `input_buf`: Buffer for storing input data.
    - `input_len`: Length of the input buffer.
    - `input_space`: Allocated space for the input buffer.
    - `input_end`: Enum indicating the end type of input (ST or BEL).
    - `param_list`: Array of input parameters.
    - `param_list_len`: Number of parameters in the list.
    - `utf8data`: Data structure for handling UTF-8 input.
    - `utf8started`: Flag indicating if UTF-8 input has started.
    - `ch`: Current character being processed.
    - `last`: Last processed UTF-8 data.
    - `flags`: Flags for input processing state.
    - `state`: Pointer to the current input state.
    - `timer`: Event timer for managing input timeouts.
    - `since_ground`: Buffer for storing input since the last ground state.
- **Description**: The `input_ctx` structure is a comprehensive context for managing input parsing and processing in a terminal emulator. It maintains the state of the input parser, including the current and previous screen states, input buffers, and parameters for escape sequences. It also handles UTF-8 character processing and manages input-related events and timers. This structure is crucial for interpreting and executing terminal control sequences, updating the screen, and managing input-related events in a terminal environment.


---
### input_table_entry
- **Type**: `struct`
- **Members**:
    - `ch`: An integer representing a character code.
    - `interm`: A pointer to a constant character string, used for intermediate characters in escape sequences.
    - `type`: An integer representing the type of input command or escape sequence.
- **Description**: The `input_table_entry` structure is used to define entries in a command table for parsing input sequences in a terminal emulator. Each entry consists of a character code (`ch`), an optional intermediate string (`interm`), and a type identifier (`type`) that specifies the kind of command or escape sequence being represented. This structure is part of a larger system for handling terminal input, allowing the program to interpret and respond to various control sequences.


---
### input_esc_type
- **Type**: `enum`
- **Members**:
    - `INPUT_ESC_DECALN`: Represents the DECALN escape sequence.
    - `INPUT_ESC_DECKPAM`: Represents the DECKPAM escape sequence.
    - `INPUT_ESC_DECKPNM`: Represents the DECKPNM escape sequence.
    - `INPUT_ESC_DECRC`: Represents the DECRC escape sequence.
    - `INPUT_ESC_DECSC`: Represents the DECSC escape sequence.
    - `INPUT_ESC_HTS`: Represents the HTS escape sequence.
    - `INPUT_ESC_IND`: Represents the IND escape sequence.
    - `INPUT_ESC_NEL`: Represents the NEL escape sequence.
    - `INPUT_ESC_RI`: Represents the RI escape sequence.
    - `INPUT_ESC_RIS`: Represents the RIS escape sequence.
    - `INPUT_ESC_SCSG0_OFF`: Represents the SCSG0_OFF escape sequence.
    - `INPUT_ESC_SCSG0_ON`: Represents the SCSG0_ON escape sequence.
    - `INPUT_ESC_SCSG1_OFF`: Represents the SCSG1_OFF escape sequence.
    - `INPUT_ESC_SCSG1_ON`: Represents the SCSG1_ON escape sequence.
    - `INPUT_ESC_ST`: Represents the ST escape sequence.
- **Description**: The `input_esc_type` is an enumeration that defines a set of constants representing various escape sequences used in terminal emulation. Each member of the enum corresponds to a specific escape sequence, such as DECALN, DECKPAM, and others, which are used to control terminal behavior and formatting. This enum is part of a larger input parsing system that interprets and processes terminal input sequences.


---
### input_csi_type
- **Type**: `enum`
- **Members**:
    - `INPUT_CSI_CBT`: Represents the CSI command for Cursor Backward Tabulation.
    - `INPUT_CSI_CNL`: Represents the CSI command for Cursor Next Line.
    - `INPUT_CSI_CPL`: Represents the CSI command for Cursor Previous Line.
    - `INPUT_CSI_CUB`: Represents the CSI command for Cursor Backward.
    - `INPUT_CSI_CUD`: Represents the CSI command for Cursor Down.
    - `INPUT_CSI_CUF`: Represents the CSI command for Cursor Forward.
    - `INPUT_CSI_CUP`: Represents the CSI command for Cursor Position.
    - `INPUT_CSI_CUU`: Represents the CSI command for Cursor Up.
    - `INPUT_CSI_DA`: Represents the CSI command for Device Attributes.
    - `INPUT_CSI_DA_TWO`: Represents the CSI command for Secondary Device Attributes.
    - `INPUT_CSI_DCH`: Represents the CSI command for Delete Character.
    - `INPUT_CSI_DECSCUSR`: Represents the CSI command for Set Cursor Style.
    - `INPUT_CSI_DECSTBM`: Represents the CSI command for Set Top and Bottom Margins.
    - `INPUT_CSI_DL`: Represents the CSI command for Delete Line.
    - `INPUT_CSI_DSR`: Represents the CSI command for Device Status Report.
    - `INPUT_CSI_DSR_PRIVATE`: Represents the CSI command for Private Device Status Report.
    - `INPUT_CSI_ECH`: Represents the CSI command for Erase Character.
    - `INPUT_CSI_ED`: Represents the CSI command for Erase in Display.
    - `INPUT_CSI_EL`: Represents the CSI command for Erase in Line.
    - `INPUT_CSI_HPA`: Represents the CSI command for Horizontal Position Absolute.
    - `INPUT_CSI_ICH`: Represents the CSI command for Insert Character.
    - `INPUT_CSI_IL`: Represents the CSI command for Insert Line.
    - `INPUT_CSI_MODOFF`: Represents the CSI command for Mode Off.
    - `INPUT_CSI_MODSET`: Represents the CSI command for Mode Set.
    - `INPUT_CSI_QUERY_PRIVATE`: Represents the CSI command for Private Query.
    - `INPUT_CSI_RCP`: Represents the CSI command for Restore Cursor Position.
    - `INPUT_CSI_REP`: Represents the CSI command for Repeat the preceding graphic character.
    - `INPUT_CSI_RM`: Represents the CSI command for Reset Mode.
    - `INPUT_CSI_RM_PRIVATE`: Represents the CSI command for Private Reset Mode.
    - `INPUT_CSI_SCP`: Represents the CSI command for Save Cursor Position.
    - `INPUT_CSI_SD`: Represents the CSI command for Scroll Down.
    - `INPUT_CSI_SGR`: Represents the CSI command for Select Graphic Rendition.
    - `INPUT_CSI_SM`: Represents the CSI command for Set Mode.
    - `INPUT_CSI_SM_GRAPHICS`: Represents the CSI command for Set Mode Graphics.
    - `INPUT_CSI_SM_PRIVATE`: Represents the CSI command for Private Set Mode.
    - `INPUT_CSI_SU`: Represents the CSI command for Scroll Up.
    - `INPUT_CSI_TBC`: Represents the CSI command for Tabulation Clear.
    - `INPUT_CSI_VPA`: Represents the CSI command for Vertical Position Absolute.
    - `INPUT_CSI_WINOPS`: Represents the CSI command for Window Operations.
    - `INPUT_CSI_XDA`: Represents the CSI command for Xterm Device Attributes.
- **Description**: The `input_csi_type` enumeration defines a set of constants representing various Control Sequence Introducer (CSI) commands used in terminal emulation. Each enumerator corresponds to a specific CSI command, which is part of the ANSI escape codes used to control cursor movement, text formatting, and other terminal functions. This enumeration is integral to parsing and handling terminal input sequences, allowing the software to interpret and execute the appropriate actions based on the received CSI commands.


---
### input_transition
- **Type**: `struct`
- **Members**:
    - `first`: Represents the first character in a range for a transition.
    - `last`: Represents the last character in a range for a transition.
    - `handler`: A function pointer to handle the transition when it occurs.
    - `state`: A pointer to the next input state after the transition.
- **Description**: The `input_transition` structure defines a state transition in an input parser, specifying a range of characters that trigger the transition, a handler function to execute when the transition occurs, and the next state to transition to. It is used in a state machine to manage input parsing, allowing for complex input sequences to be processed and handled appropriately.


---
### input_state
- **Type**: `struct`
- **Members**:
    - `name`: A constant character pointer representing the name of the input state.
    - `enter`: A function pointer to a function that is called when entering the input state.
    - `exit`: A function pointer to a function that is called when exiting the input state.
    - `transitions`: A constant pointer to an array of input_transition structures defining state transitions.
- **Description**: The `input_state` structure defines a state in an input parsing state machine, typically used for handling terminal input sequences. It includes a name for the state, function pointers for actions to perform when entering or exiting the state, and a list of transitions that define how the state machine moves from one state to another based on input.


# Functions

---
### printflike<!-- {{#callable:printflike}} -->
The `printflike` function formats and sends a formatted string to a terminal or output stream.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for the input being processed.
    - `fmt`: A format string that specifies how subsequent arguments are converted for output.
    - `...`: A variable number of additional arguments that are formatted according to the format string.
- **Control Flow**:
    - The function begins by checking if the `event` field of the `input_ctx` structure is NULL; if it is, the function returns immediately.
    - It then initializes a variable argument list using `va_start` to process the additional arguments.
    - The formatted string is created using `xvasprintf`, which allocates memory for the formatted string based on the provided format and arguments.
    - A debug log is generated to record the formatted string.
    - Finally, the formatted string is sent to the output stream using `bufferevent_write`, and the allocated memory for the formatted string is freed.
- **Output**:
    - The function does not return a value; instead, it sends the formatted output to the specified output stream associated with the `input_ctx`.


---
### input_timer_callback<!-- {{#callable:input_timer_callback}} -->
The `input_timer_callback` function handles the expiration of an input timer by logging the current state and resetting the input context.
- **Inputs**:
    - `fd`: An unused file descriptor, typically for event handling.
    - `events`: An unused event mask indicating the type of events that triggered the callback.
    - `arg`: A pointer to the `input_ctx` structure that holds the input context.
- **Control Flow**:
    - The function begins by casting the `arg` pointer to an `input_ctx` structure.
    - It logs a debug message indicating that the timer has expired along with the current state name.
    - The [`input_reset`](#input_reset) function is called to reset the input context to its initial state.
- **Output**:
    - The function does not return a value; it performs actions that affect the state of the input context and logs a message.
- **Functions called**:
    - [`input_reset`](#input_reset)


---
### input_start_timer<!-- {{#callable:input_start_timer}} -->
The `input_start_timer` function initializes and starts a timer for the input context.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
- **Control Flow**:
    - The function creates a `struct timeval` instance `tv` with a duration of 5 seconds.
    - It first calls `event_del` to remove any existing timer associated with the input context.
    - Then, it calls `event_add` to add the new timer with the specified duration.
- **Output**:
    - The function does not return a value; it modifies the state of the input context by starting a timer.


---
### input_reset_cell<!-- {{#callable:input_reset_cell}} -->
Resets the input cell state to default values.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
- **Control Flow**:
    - Copies the default cell attributes from `grid_default_cell` to the current cell in the input context.
    - Resets the `set`, `g0set`, and `g1set` flags of the current cell to 0.
    - Copies the current cell state to the `old_cell` field in the input context.
    - Resets the `old_cx` and `old_cy` coordinates to 0.
- **Output**:
    - The function does not return a value; it modifies the state of the `input_ctx` structure directly.


---
### input_save_state<!-- {{#callable:input_save_state}} -->
The `input_save_state` function saves the current state of the input context.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the current input context.
- **Control Flow**:
    - The function retrieves the current screen context from the input context.
    - It copies the current cell state from `ictx->cell` to `ictx->old_cell`.
    - It saves the current cursor x-coordinate from the screen to `ictx->old_cx`.
    - It saves the current cursor y-coordinate from the screen to `ictx->old_cy`.
    - It saves the current mode from the screen to `ictx->old_mode`.
- **Output**:
    - The function does not return a value; it modifies the state of the `input_ctx` structure to reflect the saved state.


---
### input_restore_state<!-- {{#callable:input_restore_state}} -->
Restores the previous input state in the terminal context.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context, including the current and previous cell states.
- **Control Flow**:
    - Copies the current cell state from `old_cell` to `cell` using `memcpy`.
    - Checks if the `old_mode` has the `MODE_ORIGIN` flag set.
    - If `MODE_ORIGIN` is set, it calls [`screen_write_mode_set`](screen-write.c.driver.md#screen_write_mode_set) to restore the origin mode.
    - If `MODE_ORIGIN` is not set, it calls [`screen_write_mode_clear`](screen-write.c.driver.md#screen_write_mode_clear) to clear the origin mode.
    - Moves the cursor to the previous coordinates stored in `old_cx` and `old_cy` using [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove).
- **Output**:
    - The function does not return a value; it modifies the state of the terminal's input context and screen directly.
- **Functions called**:
    - [`screen_write_mode_set`](screen-write.c.driver.md#screen_write_mode_set)
    - [`screen_write_mode_clear`](screen-write.c.driver.md#screen_write_mode_clear)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)


---
### input_init<!-- {{#callable:input_init}} -->
Initializes an `input_ctx` structure for handling input in a terminal pane.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure representing the pane where input will be processed.
    - `bev`: A pointer to a `bufferevent` structure used for asynchronous I/O operations.
    - `palette`: A pointer to a `colour_palette` structure that holds color settings for the input context.
- **Control Flow**:
    - Allocates memory for a new `input_ctx` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc).
    - Assigns the provided `window_pane`, `bufferevent`, and `colour_palette` to the corresponding fields in the `input_ctx` structure.
    - Initializes the input buffer size and allocates memory for the input buffer using [`xmalloc`](xmalloc.c.driver.md#xmalloc).
    - Creates a new event buffer for storing input received since the last ground state using `evbuffer_new`.
    - Sets a timer for input handling using `evtimer_set` with a callback to `input_timer_callback`.
    - Calls [`input_reset`](#input_reset) to initialize the input context to a default state.
    - Returns a pointer to the initialized `input_ctx` structure.
- **Output**:
    - Returns a pointer to the newly initialized `input_ctx` structure.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`input_reset`](#input_reset)


---
### input_free<!-- {{#callable:input_free}} -->
The `input_free` function deallocates resources associated with an `input_ctx` structure.
- **Inputs**:
    - `ictx`: A pointer to an `input_ctx` structure that holds the context for input processing.
- **Control Flow**:
    - Iterates over the `param_list` of the `input_ctx` to free any allocated strings of type `INPUT_STRING`.
    - Calls `event_del` to remove the associated timer event from the event loop.
    - Frees the `input_buf` buffer allocated for input data.
    - Frees the `since_ground` buffer which holds input received since the last ground state.
    - Finally, frees the `input_ctx` structure itself.
- **Output**:
    - The function does not return a value; it performs cleanup and deallocation of resources.


---
### input_reset<!-- {{#callable:input_reset}} -->
Resets the input context state and optionally clears the screen.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
    - `clear`: An integer flag indicating whether to clear the screen (1 to clear, 0 otherwise).
- **Control Flow**:
    - Calls [`input_reset_cell`](#input_reset_cell) to reset the input cell state.
    - If `clear` is true and the associated `window_pane` is not NULL, it checks if the pane modes are empty and starts the screen writing context accordingly.
    - Calls [`screen_write_reset`](screen-write.c.driver.md#screen_write_reset) to reset the screen writing context.
    - Calls [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop) to stop the screen writing context.
    - Calls [`input_clear`](#input_clear) to clear the input context.
    - Sets the input context state to `input_state_ground` and resets the flags.
- **Output**:
    - The function does not return a value; it modifies the state of the input context and the screen as needed.
- **Functions called**:
    - [`input_reset_cell`](#input_reset_cell)
    - [`screen_write_start_pane`](screen-write.c.driver.md#screen_write_start_pane)
    - [`screen_write_start`](screen-write.c.driver.md#screen_write_start)
    - [`screen_write_reset`](screen-write.c.driver.md#screen_write_reset)
    - [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop)
    - [`input_clear`](#input_clear)


---
### input_pending<!-- {{#callable:input_pending}} -->
The `input_pending` function retrieves the buffer containing all input received since the last ground state.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
- **Control Flow**:
    - The function directly accesses the `since_ground` member of the `input_ctx` structure.
    - It returns the value of `since_ground`, which is a pointer to an `evbuffer` containing the pending input.
- **Output**:
    - Returns a pointer to an `evbuffer` that contains all input received since the last ground state.


---
### input_set_state<!-- {{#callable:input_set_state}} -->
The `input_set_state` function changes the current state of the input context by executing the exit handler of the current state and the enter handler of the new state.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure representing the current input context.
    - `itr`: A pointer to the `input_transition` structure that contains the new state to transition to.
- **Control Flow**:
    - Check if the current state's exit handler is not NULL and call it with the input context.
    - Update the current state of the input context to the new state specified in the transition.
    - Check if the new state's enter handler is not NULL and call it with the input context.
- **Output**:
    - The function does not return a value; it modifies the state of the input context directly.


---
### input_parse<!-- {{#callable:input_parse}} -->
The `input_parse` function processes a buffer of input data, transitioning between different input states based on the characters read.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input parsing.
    - `buf`: A pointer to a buffer containing the input data to be parsed.
    - `len`: The length of the input data in the buffer.
- **Control Flow**:
    - The function initializes a loop that continues until all input data has been processed.
    - For each character in the input buffer, it checks if a state transition is needed based on the current state and the character.
    - If a transition is required, it searches through the available transitions for the current state to find a matching handler.
    - If a handler is found, it is executed; if it returns non-zero, the state does not change.
    - If the handler indicates a state change, the new state is set.
    - If the current state is not the ground state, the character is added to the `since_ground` buffer.
- **Output**:
    - The function does not return a value; instead, it modifies the state of the `input_ctx` and may produce side effects such as updating the screen or collecting input data.
- **Functions called**:
    - [`screen_write_collect_end`](screen-write.c.driver.md#screen_write_collect_end)
    - [`input_set_state`](#input_set_state)


---
### input_parse_pane<!-- {{#callable:input_parse_pane}} -->
Parses input data from a `window_pane` and updates its state.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure that contains the state and data to be parsed.
- **Control Flow**:
    - Calls [`window_pane_get_new_data`](window.c.driver.md#window_pane_get_new_data) to retrieve new input data and its size from the `window_pane`.
    - Invokes [`input_parse_buffer`](#input_parse_buffer) to process the retrieved data.
    - Updates the `window_pane`'s offset using [`window_pane_update_used_data`](window.c.driver.md#window_pane_update_used_data) to reflect the amount of data that has been processed.
- **Output**:
    - The function does not return a value; it modifies the state of the `window_pane` based on the parsed input.
- **Functions called**:
    - [`window_pane_get_new_data`](window.c.driver.md#window_pane_get_new_data)
    - [`input_parse_buffer`](#input_parse_buffer)
    - [`window_pane_update_used_data`](window.c.driver.md#window_pane_update_used_data)


---
### input_parse_buffer<!-- {{#callable:input_parse_buffer}} -->
Parses input data from a buffer for a specific window pane.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane that is receiving the input.
    - `buf`: A pointer to a buffer containing the input data to be parsed.
    - `len`: The length of the input data in bytes.
- **Control Flow**:
    - If the length of the input data is zero, the function returns immediately.
    - The function updates the activity status of the window associated with the pane.
    - It sets the pane's flags to indicate that it has changed and may have unseen changes if it is in a mode.
    - Depending on whether there are modes active, it either starts writing to the pane or the screen.
    - The function logs the input data for debugging purposes.
    - It calls the [`input_parse`](#input_parse) function to handle the actual parsing of the input data.
    - Finally, it stops the screen writing context.
- **Output**:
    - The function does not return a value; it modifies the state of the window pane and processes the input data.
- **Functions called**:
    - [`window_update_activity`](window.c.driver.md#window_update_activity)
    - [`screen_write_start_pane`](screen-write.c.driver.md#screen_write_start_pane)
    - [`screen_write_start`](screen-write.c.driver.md#screen_write_start)
    - [`input_parse`](#input_parse)
    - [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop)


---
### input_parse_screen<!-- {{#callable:input_parse_screen}} -->
Parses input data for a screen context and processes it accordingly.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input parsing.
    - `s`: A pointer to the `screen` structure that represents the screen being written to.
    - `cb`: A callback function of type `screen_write_init_ctx_cb` used to initialize the screen write context.
    - `arg`: A pointer to user-defined data passed to the callback function.
    - `buf`: A pointer to a buffer containing the input data to be parsed.
    - `len`: The length of the input data in the buffer.
- **Control Flow**:
    - Checks if the length of the input data is zero; if so, it returns immediately.
    - Calls [`screen_write_start_callback`](screen-write.c.driver.md#screen_write_start_callback) to initialize the screen write context with the provided callback and argument.
    - Calls [`input_parse`](#input_parse) to process the input data from the buffer.
    - Finally, calls [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop) to finalize the screen writing process.
- **Output**:
    - The function does not return a value; it modifies the screen context based on the parsed input.
- **Functions called**:
    - [`screen_write_start_callback`](screen-write.c.driver.md#screen_write_start_callback)
    - [`input_parse`](#input_parse)
    - [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop)


---
### input_split<!-- {{#callable:input_split}} -->
The `input_split` function processes a buffer of input parameters, categorizing them as strings or numbers and storing them in a context structure.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input parsing, including the buffer of parameters to be split.
- **Control Flow**:
    - The function begins by freeing any previously allocated strings in the `param_list` of the `input_ctx` structure.
    - It resets the `param_list_len` to zero and checks if the `param_len` is zero, returning early if true.
    - It initializes a pointer to the first parameter in the `param_list` and starts processing the `param_buf` using [`strsep`](compat/strsep.c.driver.md#strsep) to split it by semicolons.
    - For each split parameter, it checks if it is empty, marking it as `INPUT_MISSING`, or checks for a colon to determine if it is a string or a number.
    - If it is a string, it duplicates the string and assigns it to the `str` field of the `input_param` structure; if it is a number, it converts it using [`strtonum`](compat/strtonum.c.driver.md#strtonum).
    - If the conversion fails, it returns -1 to indicate an error.
    - After processing all parameters, it logs the type and value of each parameter in the `param_list`.
- **Output**:
    - The function returns 0 on success, or -1 if an error occurs during number conversion or if the parameter list exceeds its allocated size.
- **Functions called**:
    - [`strsep`](compat/strsep.c.driver.md#strsep)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)


---
### input_get<!-- {{#callable:input_get}} -->
The `input_get` function retrieves a parameter value from the input context, ensuring it meets specified constraints.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input parameters.
    - `validx`: An unsigned integer representing the index of the parameter to retrieve.
    - `minval`: An integer specifying the minimum acceptable value for the parameter.
    - `defval`: An integer that serves as the default value to return if the parameter is invalid.
- **Control Flow**:
    - Checks if `validx` is greater than or equal to the length of the parameter list; if so, returns `defval`.
    - Retrieves the parameter at the specified index from the parameter list.
    - If the parameter type is `INPUT_MISSING`, returns `defval`.
    - If the parameter type is `INPUT_STRING`, returns -1 to indicate an error.
    - Checks if the retrieved numeric value is less than `minval`; if so, returns `minval`.
    - Returns the retrieved numeric value if all checks pass.
- **Output**:
    - Returns the parameter value if valid, `minval` if the value is below the minimum, or `defval` if the parameter is missing or invalid.


---
### input_reply<!-- {{#callable:input_reply}} -->
The `input_reply` function formats and sends a reply message to a terminal using a specified format.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for the input handling.
    - `fmt`: A format string that specifies how to format the reply message.
    - `...`: Additional arguments that are formatted according to the format string.
- **Control Flow**:
    - Checks if the `event` member of `ictx` is NULL; if so, the function returns immediately.
    - Initializes a variable argument list using `va_start` with `fmt`.
    - Calls [`xvasprintf`](xmalloc.c.driver.md#xvasprintf) to format the reply message into a dynamically allocated string.
    - Logs the formatted reply message using `log_debug`.
    - Writes the formatted reply message to the buffer event using `bufferevent_write`.
    - Frees the dynamically allocated reply string.
- **Output**:
    - The function does not return a value; it sends a formatted reply message to the terminal.
- **Functions called**:
    - [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)


---
### input_clear<!-- {{#callable:input_clear}} -->
The `input_clear` function resets the input context by clearing buffers and flags.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the state and buffers for input processing.
- **Control Flow**:
    - The function begins by deleting any active timer associated with the input context.
    - It then clears the intermediate buffer by setting its first character to null and resetting its length.
    - The parameter buffer is similarly cleared and its length reset.
    - The input buffer is cleared, and its length is reset to zero.
    - The input end state is set to `INPUT_END_ST`.
    - Finally, the `INPUT_DISCARD` flag is cleared from the context's flags.
- **Output**:
    - The function does not return a value; it modifies the state of the `input_ctx` structure directly.


---
### input_ground<!-- {{#callable:input_ground}} -->
The `input_ground` function resets the input context to its ground state.
- **Inputs**:
    - None
- **Control Flow**:
    - The function begins by deleting any existing timer associated with the input context.
    - It then drains the buffer of input received since the last ground state.
    - If the input space exceeds the defined starting buffer size, it resets the input space to the starting size and reallocates the input buffer accordingly.
- **Output**:
    - The function does not return a value; it modifies the state of the input context directly.
- **Functions called**:
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)


---
### input_print<!-- {{#callable:input_print}} -->
The `input_print` function processes a character input and updates the screen context accordingly.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context, including the character to be printed and the screen write context.
- **Control Flow**:
    - The function initializes the `utf8started` flag to 0, indicating that the current character cannot be valid UTF-8.
    - It determines the character set to use based on the `set` value of the `cell` in the `input_ctx` structure.
    - If the character set is set to 1, it updates the cell's attributes to include `GRID_ATTR_CHARSET`, otherwise it clears this attribute.
    - The character is then set in the `cell.data` using the [`utf8_set`](utf8.c.driver.md#utf8_set) function.
    - The updated cell is added to the screen write context using [`screen_write_collect_add`](screen-write.c.driver.md#screen_write_collect_add).
    - The last character data is copied to `last` for potential future use.
    - The `INPUT_LAST` flag is set in the `flags` of the `input_ctx` to indicate that the last character has been processed.
    - Finally, the `GRID_ATTR_CHARSET` attribute is cleared from the cell.
- **Output**:
    - The function returns 0, indicating successful processing of the input character.
- **Functions called**:
    - [`utf8_set`](utf8.c.driver.md#utf8_set)
    - [`screen_write_collect_add`](screen-write.c.driver.md#screen_write_collect_add)
    - [`utf8_copy`](utf8.c.driver.md#utf8_copy)


---
### input_intermediate<!-- {{#callable:input_intermediate}} -->
The `input_intermediate` function collects intermediate input characters into a buffer.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
- **Control Flow**:
    - Checks if the length of the intermediate buffer has reached its maximum capacity minus one.
    - If the buffer is full, it sets the `INPUT_DISCARD` flag in the context's flags.
    - If there is space in the buffer, it appends the current character to the buffer and increments the buffer length.
    - Finally, it null-terminates the string in the buffer.
- **Output**:
    - Returns 0 to indicate successful execution.


---
### input_parameter<!-- {{#callable:input_parameter}} -->
The `input_parameter` function collects a character input into a parameter buffer and sets a discard flag if the buffer is full.
- **Inputs**:
    - `ictx`: A pointer to a `struct input_ctx` that contains the context for input parsing, including the parameter buffer and its length.
- **Control Flow**:
    - Checks if the current length of the parameter buffer equals its maximum size minus one.
    - If the buffer is full, it sets the `INPUT_DISCARD` flag in the context's flags.
    - If there is space in the buffer, it adds the current character to the buffer, increments the length, and null-terminates the string.
- **Output**:
    - Returns 0 to indicate successful execution.


---
### input_input<!-- {{#callable:input_input}} -->
The `input_input` function appends a character to the input buffer, dynamically resizing the buffer if necessary.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing, including the input buffer and its properties.
- **Control Flow**:
    - The function retrieves the current size of the input buffer from `ictx->input_space`.
    - It enters a loop that checks if the current length of the input plus one (for the null terminator) exceeds the available space.
    - If the buffer is full, it doubles the size of the buffer until it is either large enough or exceeds a predefined maximum size.
    - If the new size exceeds `input_buffer_size`, it sets a discard flag in `ictx->flags` and exits the function.
    - If there is space, it appends the character `ictx->ch` to the buffer, increments the input length, and null-terminates the string.
- **Output**:
    - The function returns 0 upon successful execution, indicating that the character has been added to the input buffer.
- **Functions called**:
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)


---
### input_c0_dispatch<!-- {{#callable:input_c0_dispatch}} -->
The `input_c0_dispatch` function processes control characters in the input context.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input state and context.
- **Control Flow**:
    - The function starts by resetting the UTF-8 state and logging the received character.
    - A switch statement is used to handle different control characters (C0 controls).
    - For each case, specific actions are taken, such as handling backspace, tab, line feed, carriage return, and others.
    - The function modifies the cursor position or screen state based on the control character processed.
    - Finally, it clears the INPUT_LAST flag and returns 0.
- **Output**:
    - The function returns 0, indicating successful processing of the input character.
- **Functions called**:
    - [`alerts_queue`](alerts.c.driver.md#alerts_queue)
    - [`screen_write_backspace`](screen-write.c.driver.md#screen_write_backspace)
    - [`grid_get_cell`](grid.c.driver.md#grid_get_cell)
    - [`grid_cells_look_equal`](grid.c.driver.md#grid_cells_look_equal)
    - [`grid_set_tab`](grid.c.driver.md#grid_set_tab)
    - [`screen_write_collect_add`](screen-write.c.driver.md#screen_write_collect_add)
    - [`screen_write_linefeed`](screen-write.c.driver.md#screen_write_linefeed)
    - [`screen_write_carriagereturn`](screen-write.c.driver.md#screen_write_carriagereturn)


---
### input_esc_dispatch<!-- {{#callable:input_esc_dispatch}} -->
The `input_esc_dispatch` function processes escape sequences in the input context.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context, including state and parameters.
- **Control Flow**:
    - Checks if the `INPUT_DISCARD` flag is set in the `ictx` structure; if so, it returns immediately.
    - Logs the current character and intermediate buffer for debugging purposes.
    - Performs a binary search on the `input_esc_table` to find a matching escape sequence entry based on the current character.
    - If no matching entry is found, it logs an unknown escape sequence and returns.
    - Based on the type of the matched entry, it executes specific actions such as resetting the screen, handling line feeds, or modifying the input context state.
    - Clears the `INPUT_LAST` flag in the `ictx` structure before returning.
- **Output**:
    - The function returns 0 upon successful processing of the escape sequence, or it may return early without performing any actions if the `INPUT_DISCARD` flag is set or if the escape sequence is unknown.
- **Functions called**:
    - [`colour_palette_clear`](colour.c.driver.md#colour_palette_clear)
    - [`input_reset_cell`](#input_reset_cell)
    - [`screen_write_reset`](screen-write.c.driver.md#screen_write_reset)
    - [`screen_write_fullredraw`](screen-write.c.driver.md#screen_write_fullredraw)
    - [`screen_write_linefeed`](screen-write.c.driver.md#screen_write_linefeed)
    - [`screen_write_carriagereturn`](screen-write.c.driver.md#screen_write_carriagereturn)
    - [`screen_write_reverseindex`](screen-write.c.driver.md#screen_write_reverseindex)
    - [`screen_write_mode_set`](screen-write.c.driver.md#screen_write_mode_set)
    - [`screen_write_mode_clear`](screen-write.c.driver.md#screen_write_mode_clear)
    - [`input_save_state`](#input_save_state)
    - [`input_restore_state`](#input_restore_state)
    - [`screen_write_alignmenttest`](screen-write.c.driver.md#screen_write_alignmenttest)


---
### input_csi_dispatch<!-- {{#callable:input_csi_dispatch}} -->
The `input_csi_dispatch` function processes Control Sequence Introducer (CSI) commands for terminal input.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context, including state, parameters, and screen context.
- **Control Flow**:
    - Checks if the `INPUT_DISCARD` flag is set in `ictx`, returning early if true.
    - Logs the current character and intermediate buffers for debugging.
    - Calls [`input_split`](#input_split) to parse parameters from the input context.
    - Uses `bsearch` to find the corresponding entry in the `input_csi_table` based on the current character.
    - If no entry is found, logs an unknown character and returns.
    - Processes the command based on the type of the entry found, executing different actions such as cursor movement, screen clearing, or mode setting.
    - Handles various CSI commands like cursor movement (CUP, CUB, CUD, CUF), screen clearing (ED, EL), and more.
    - Each command may involve further calls to helper functions to perform specific screen operations.
- **Output**:
    - The function returns 0 upon successful processing of the CSI command, or it may return early with no output if the command is discarded or unrecognized.
- **Functions called**:
    - [`input_split`](#input_split)
    - [`input_get`](#input_get)
    - [`screen_write_cursorleft`](screen-write.c.driver.md#screen_write_cursorleft)
    - [`screen_write_cursordown`](screen-write.c.driver.md#screen_write_cursordown)
    - [`screen_write_cursorright`](screen-write.c.driver.md#screen_write_cursorright)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`screen_write_mode_clear`](screen-write.c.driver.md#screen_write_mode_clear)
    - [`screen_write_mode_set`](screen-write.c.driver.md#screen_write_mode_set)
    - [`input_csi_dispatch_winops`](#input_csi_dispatch_winops)
    - [`screen_write_cursorup`](screen-write.c.driver.md#screen_write_cursorup)
    - [`screen_write_carriagereturn`](screen-write.c.driver.md#screen_write_carriagereturn)
    - [`input_reply`](#input_reply)
    - [`screen_write_clearcharacter`](screen-write.c.driver.md#screen_write_clearcharacter)
    - [`screen_write_deletecharacter`](screen-write.c.driver.md#screen_write_deletecharacter)
    - [`screen_write_scrollregion`](screen-write.c.driver.md#screen_write_scrollregion)
    - [`screen_write_deleteline`](screen-write.c.driver.md#screen_write_deleteline)
    - [`input_report_current_theme`](#input_report_current_theme)
    - [`screen_write_clearendofscreen`](screen-write.c.driver.md#screen_write_clearendofscreen)
    - [`screen_write_clearstartofscreen`](screen-write.c.driver.md#screen_write_clearstartofscreen)
    - [`screen_write_clearscreen`](screen-write.c.driver.md#screen_write_clearscreen)
    - [`screen_write_clearhistory`](screen-write.c.driver.md#screen_write_clearhistory)
    - [`screen_write_clearendofline`](screen-write.c.driver.md#screen_write_clearendofline)
    - [`screen_write_clearstartofline`](screen-write.c.driver.md#screen_write_clearstartofline)
    - [`screen_write_clearline`](screen-write.c.driver.md#screen_write_clearline)
    - [`screen_write_insertcharacter`](screen-write.c.driver.md#screen_write_insertcharacter)
    - [`screen_write_insertline`](screen-write.c.driver.md#screen_write_insertline)
    - [`utf8_copy`](utf8.c.driver.md#utf8_copy)
    - [`screen_write_collect_add`](screen-write.c.driver.md#screen_write_collect_add)
    - [`input_restore_state`](#input_restore_state)
    - [`input_csi_dispatch_rm`](#input_csi_dispatch_rm)
    - [`input_csi_dispatch_rm_private`](#input_csi_dispatch_rm_private)
    - [`input_save_state`](#input_save_state)
    - [`input_csi_dispatch_sgr`](#input_csi_dispatch_sgr)
    - [`input_csi_dispatch_sm`](#input_csi_dispatch_sm)
    - [`input_csi_dispatch_sm_private`](#input_csi_dispatch_sm_private)
    - [`input_csi_dispatch_sm_graphics`](#input_csi_dispatch_sm_graphics)
    - [`screen_write_scrollup`](screen-write.c.driver.md#screen_write_scrollup)
    - [`screen_write_scrolldown`](screen-write.c.driver.md#screen_write_scrolldown)
    - [`screen_set_cursor_style`](screen.c.driver.md#screen_set_cursor_style)
    - [`getversion`](tmux.c.driver.md#getversion)


---
### input_csi_dispatch_rm<!-- {{#callable:input_csi_dispatch_rm}} -->
The `input_csi_dispatch_rm` function processes Control Sequence Introducer (CSI) 'Reset Mode' commands.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input processing, including the current screen write context and parameter list.
- **Control Flow**:
    - Iterates over the parameters in the `param_list_len` of the `input_ctx` structure.
    - For each parameter, it retrieves the value using the [`input_get`](#input_get) function.
    - Based on the retrieved value, it performs specific actions such as clearing the insert mode or setting the cursor visibility.
    - If an unknown parameter is encountered, it logs a debug message.
- **Output**:
    - The function does not return a value but modifies the screen state based on the parameters processed, such as clearing modes or setting cursor visibility.
- **Functions called**:
    - [`input_get`](#input_get)
    - [`screen_write_mode_clear`](screen-write.c.driver.md#screen_write_mode_clear)
    - [`screen_write_mode_set`](screen-write.c.driver.md#screen_write_mode_set)


---
### input_csi_dispatch_rm_private<!-- {{#callable:input_csi_dispatch_rm_private}} -->
The `input_csi_dispatch_rm_private` function processes private mode reset (RM) control sequences for terminal input.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing, including the current state and parameters.
- **Control Flow**:
    - Iterates over the `param_list_len` of the `input_ctx` to process each parameter.
    - For each parameter, it retrieves its value using the [`input_get`](#input_get) function.
    - Based on the retrieved value, it executes specific screen writing functions to clear or set various modes.
    - If an unknown parameter is encountered, it logs a debug message.
- **Output**:
    - The function does not return a value but modifies the screen state by clearing or setting various modes based on the parameters received.
- **Functions called**:
    - [`input_get`](#input_get)
    - [`screen_write_mode_clear`](screen-write.c.driver.md#screen_write_mode_clear)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`screen_write_clearscreen`](screen-write.c.driver.md#screen_write_clearscreen)
    - [`screen_write_mode_set`](screen-write.c.driver.md#screen_write_mode_set)
    - [`screen_write_alternateoff`](screen-write.c.driver.md#screen_write_alternateoff)


---
### input_csi_dispatch_sm<!-- {{#callable:input_csi_dispatch_sm}} -->
The `input_csi_dispatch_sm` function processes Control Sequence Introducer (CSI) 'Set Mode' commands for terminal input.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing, including the current state and parameters.
- **Control Flow**:
    - Iterates over the parameters in the `param_list_len` of the `input_ctx` structure.
    - For each parameter, it retrieves its value using the [`input_get`](#input_get) function.
    - Based on the retrieved value, it executes specific actions such as setting or clearing modes using [`screen_write_mode_set`](screen-write.c.driver.md#screen_write_mode_set) or [`screen_write_mode_clear`](screen-write.c.driver.md#screen_write_mode_clear).
    - If an unknown parameter is encountered, it logs a debug message.
- **Output**:
    - The function does not return a value but modifies the terminal's state based on the processed CSI commands.
- **Functions called**:
    - [`input_get`](#input_get)
    - [`screen_write_mode_set`](screen-write.c.driver.md#screen_write_mode_set)
    - [`screen_write_mode_clear`](screen-write.c.driver.md#screen_write_mode_clear)


---
### input_csi_dispatch_sm_private<!-- {{#callable:input_csi_dispatch_sm_private}} -->
The `input_csi_dispatch_sm_private` function processes private mode setting commands received in a Control Sequence Introducer (CSI) context.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context, including parameters and state.
- **Control Flow**:
    - Iterates over the `param_list_len` of the `input_ctx` to process each parameter.
    - For each parameter, it retrieves the value using the [`input_get`](#input_get) function.
    - Based on the retrieved value, it executes specific screen writing functions to set modes or clear modes.
    - If an unknown parameter is encountered, it logs a debug message.
- **Output**:
    - The function does not return a value but modifies the screen state based on the parameters processed, affecting cursor visibility, screen clearing, and other terminal modes.
- **Functions called**:
    - [`input_get`](#input_get)
    - [`screen_write_mode_set`](screen-write.c.driver.md#screen_write_mode_set)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`screen_write_clearscreen`](screen-write.c.driver.md#screen_write_clearscreen)
    - [`screen_write_mode_clear`](screen-write.c.driver.md#screen_write_mode_clear)
    - [`screen_write_alternateon`](screen-write.c.driver.md#screen_write_alternateon)


---
### input_csi_dispatch_sm_graphics<!-- {{#callable:input_csi_dispatch_sm_graphics}} -->
The `input_csi_dispatch_sm_graphics` function handles the Control Sequence Introducer (CSI) for setting graphics modes in a terminal.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input processing.
- **Control Flow**:
    - Checks if the `ENABLE_SIXEL` macro is defined to ensure that sixel graphics support is enabled.
    - Validates the length of the `param_list_len` in the `input_ctx` structure; if it exceeds 3, the function returns early.
    - Retrieves the first three parameters from the `input_ctx` using the [`input_get`](#input_get) function.
    - If the first parameter `n` is 1 and the second parameter `m` is either 1, 2, or 4, it sends a reply indicating the number of sixel color registers.
    - Otherwise, it sends a different reply based on the first parameter `n` and the third parameter `o`.
- **Output**:
    - The function sends a response back to the terminal using the [`input_reply`](#input_reply) function, which includes either the number of sixel color registers or a different response based on the parameters.
- **Functions called**:
    - [`input_get`](#input_get)
    - [`input_reply`](#input_reply)


---
### input_csi_dispatch_winops<!-- {{#callable:input_csi_dispatch_winops}} -->
The `input_csi_dispatch_winops` function processes window operations commands received in a Control Sequence Introducer (CSI) format.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input processing, including the current window pane and screen write context.
- **Control Flow**:
    - The function retrieves the current window associated with the input context if it exists.
    - It enters a loop to read input parameters using the [`input_get`](#input_get) function until no more valid parameters are found.
    - For each parameter, it uses a switch statement to handle specific window operation commands based on the parameter value.
    - Certain commands may require additional parameters, which are fetched using [`input_get`](#input_get) and processed accordingly.
    - The function sends replies back to the terminal using [`input_reply`](#input_reply) for specific commands that require feedback.
- **Output**:
    - The function does not return a value but sends responses to the terminal based on the window operations commands processed, such as window size and title changes.
- **Functions called**:
    - [`input_get`](#input_get)
    - [`input_reply`](#input_reply)
    - [`screen_push_title`](screen.c.driver.md#screen_push_title)
    - [`screen_pop_title`](screen.c.driver.md#screen_pop_title)
    - [`notify_pane`](notify.c.driver.md#notify_pane)
    - [`server_redraw_window_borders`](server-fn.c.driver.md#server_redraw_window_borders)
    - [`server_status_window`](server-fn.c.driver.md#server_status_window)


---
### input_csi_dispatch_sgr_256_do<!-- {{#callable:input_csi_dispatch_sgr_256_do}} -->
The `input_csi_dispatch_sgr_256_do` function processes 256-color SGR (Select Graphic Rendition) commands to set foreground, background, or underline colors.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
    - `fgbg`: An integer indicating whether the command is for foreground (38) or background (48) color.
    - `c`: An integer representing the color index, which can range from 0 to 255, or -1 to reset the color.
- **Control Flow**:
    - The function first checks if the color index `c` is -1 or greater than 255.
    - If `c` is -1, it sets the foreground or background color to a default value (8) based on the `fgbg` parameter.
    - If `c` is valid (0 to 255), it sets the foreground or background color to the specified color index with a 256-color flag based on the `fgbg` parameter.
    - The function returns 1 to indicate successful processing.
- **Output**:
    - The function does not return a color value but modifies the `grid_cell` structure within the `input_ctx` to reflect the new color settings.


---
### input_csi_dispatch_sgr_256<!-- {{#callable:input_csi_dispatch_sgr_256}} -->
The `input_csi_dispatch_sgr_256` function processes a Control Sequence Introducer (CSI) command for setting 256-color foreground or background attributes.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
    - `fgbg`: An integer indicating whether the command is for foreground (38) or background (48) color.
    - `i`: A pointer to an unsigned integer that serves as an index for parsing the input parameters.
- **Control Flow**:
    - The function retrieves the next color value from the input context using [`input_get`](#input_get).
    - If the retrieved color value is valid, it calls [`input_csi_dispatch_sgr_256_do`](#input_csi_dispatch_sgr_256_do) to apply the color setting.
    - If the color setting is successfully applied, it increments the index pointed to by `i`.
- **Output**:
    - The function does not return a value but modifies the color attributes of the `input_ctx` based on the parsed input.
- **Functions called**:
    - [`input_get`](#input_get)
    - [`input_csi_dispatch_sgr_256_do`](#input_csi_dispatch_sgr_256_do)


---
### input_csi_dispatch_sgr_rgb_do<!-- {{#callable:input_csi_dispatch_sgr_rgb_do}} -->
The `input_csi_dispatch_sgr_rgb_do` function processes RGB color values for foreground, background, or underline attributes in a terminal context.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
    - `fgbg`: An integer indicating whether the color is for foreground (38), background (48), or underline (58).
    - `r`: An integer representing the red component of the RGB color, expected to be in the range 0-255.
    - `g`: An integer representing the green component of the RGB color, expected to be in the range 0-255.
    - `b`: An integer representing the blue component of the RGB color, expected to be in the range 0-255.
- **Control Flow**:
    - The function first checks if the red, green, and blue values are within the valid range (0-255). If any value is out of range, it returns 0.
    - If the `fgbg` parameter indicates foreground (38), it sets the foreground color of the grid cell using the [`colour_join_rgb`](colour.c.driver.md#colour_join_rgb) function.
    - If the `fgbg` parameter indicates background (48), it sets the background color of the grid cell.
    - If the `fgbg` parameter indicates underline (58), it sets the underline color of the grid cell.
    - Finally, the function returns 1 to indicate successful processing.
- **Output**:
    - The function returns an integer: 0 if the RGB values are invalid, and 1 if the color attributes were successfully set.
- **Functions called**:
    - [`colour_join_rgb`](colour.c.driver.md#colour_join_rgb)


---
### input_csi_dispatch_sgr_rgb<!-- {{#callable:input_csi_dispatch_sgr_rgb}} -->
The `input_csi_dispatch_sgr_rgb` function processes RGB color settings for terminal graphics.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
    - `fgbg`: An integer indicating whether the RGB values are for foreground (38) or background (48) color.
    - `i`: A pointer to an unsigned integer that serves as an index for retrieving RGB values from the input parameters.
- **Control Flow**:
    - The function retrieves the red, green, and blue color values from the input context using the [`input_get`](#input_get) function.
    - It checks if the RGB values are valid (between 0 and 255) and calls [`input_csi_dispatch_sgr_rgb_do`](#input_csi_dispatch_sgr_rgb_do) to apply the colors.
    - If the color setting is successful, it increments the index pointer `i` to move to the next parameter.
- **Output**:
    - The function does not return a value but modifies the color attributes of the `grid_cell` in the input context based on the RGB values provided.
- **Functions called**:
    - [`input_get`](#input_get)
    - [`input_csi_dispatch_sgr_rgb_do`](#input_csi_dispatch_sgr_rgb_do)


---
### input_csi_dispatch_sgr_colon<!-- {{#callable:input_csi_dispatch_sgr_colon}} -->
The `input_csi_dispatch_sgr_colon` function processes SGR (Select Graphic Rendition) parameters separated by colons to modify text attributes in a terminal.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context, including the state and parameters.
    - `i`: An unsigned integer representing the index of the parameter in the `param_list` that contains the SGR parameters.
- **Control Flow**:
    - Initializes an array `p` to hold parsed parameters, setting all values to -1.
    - Duplicates the string parameter from `param_list[i]` and splits it by colons into individual components.
    - For each component, it converts the string to an integer and stores it in the `p` array, checking for errors.
    - If the first parameter is 4, it processes the second parameter to set various underscore attributes.
    - If the first parameter is 38, 48, or 58, it checks the second parameter to determine if RGB or 256 color settings should be applied.
    - Calls helper functions to handle RGB or 256 color settings based on the parsed parameters.
- **Output**:
    - The function modifies the attributes of the `grid_cell` structure in the `input_ctx`, affecting how text is rendered in the terminal based on the parsed SGR parameters.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`strsep`](compat/strsep.c.driver.md#strsep)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
    - [`input_csi_dispatch_sgr_rgb_do`](#input_csi_dispatch_sgr_rgb_do)
    - [`input_csi_dispatch_sgr_256_do`](#input_csi_dispatch_sgr_256_do)


---
### input_csi_dispatch_sgr<!-- {{#callable:input_csi_dispatch_sgr}} -->
The `input_csi_dispatch_sgr` function processes SGR (Select Graphic Rendition) control sequences to modify text attributes and colors in a terminal.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context, including the state of the terminal and the parameters for the SGR command.
- **Control Flow**:
    - Checks if the `param_list_len` is zero; if so, it resets the current cell to the default cell attributes.
    - Iterates over the parameters in the `param_list` to determine the SGR commands to execute.
    - For each parameter, it checks if it is a string or a number and processes it accordingly.
    - Handles special cases for foreground and background color settings (38, 48, 58) by calling specific functions for RGB and 256 color modes.
    - Modifies the attributes of the grid cell based on the SGR parameters, such as setting colors, brightness, and other text attributes.
- **Output**:
    - The function modifies the attributes of the `grid_cell` structure within the `input_ctx`, affecting how text is rendered in the terminal, but does not return a value.
- **Functions called**:
    - [`input_csi_dispatch_sgr_colon`](#input_csi_dispatch_sgr_colon)
    - [`input_get`](#input_get)
    - [`input_csi_dispatch_sgr_rgb`](#input_csi_dispatch_sgr_rgb)
    - [`input_csi_dispatch_sgr_256`](#input_csi_dispatch_sgr_256)


---
### input_end_bel<!-- {{#callable:input_end_bel}} -->
The `input_end_bel` function sets the input end state to `INPUT_END_BEL` and logs the function call.
- **Inputs**:
    - None
- **Control Flow**:
    - The function begins by logging the function name using `log_debug`.
    - It then sets the `input_end` member of the `input_ctx` structure to `INPUT_END_BEL`.
    - Finally, it returns 0 to indicate successful execution.
- **Output**:
    - The function returns an integer value of 0, indicating successful completion of the operation.


---
### input_enter_dcs<!-- {{#callable:input_enter_dcs}} -->
The `input_enter_dcs` function initializes the input context for a Device Control String (DCS) sequence.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
- **Control Flow**:
    - Logs the function entry using `log_debug`.
    - Calls [`input_clear`](#input_clear) to reset the input context state.
    - Starts a timer for the input context using [`input_start_timer`](#input_start_timer).
    - Clears the `INPUT_LAST` flag from the `flags` member of the input context.
- **Output**:
    - The function does not return a value; it modifies the state of the input context to prepare for processing a DCS sequence.
- **Functions called**:
    - [`input_clear`](#input_clear)
    - [`input_start_timer`](#input_start_timer)


---
### input_dcs_dispatch<!-- {{#callable:input_dcs_dispatch}} -->
The `input_dcs_dispatch` function processes Device Control String (DCS) input sequences for terminal emulation.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input processing, including the input buffer and associated window pane.
- **Control Flow**:
    - Checks if the associated window pane (`wp`) is NULL; if so, it returns 0 immediately.
    - If the `INPUT_DISCARD` flag is set in `ictx->flags`, it logs a debug message and returns 0.
    - If the first byte of the input buffer is 'q' and there are no intermediate characters, it attempts to parse a sixel image if `ENABLE_SIXEL` is defined.
    - Retrieves the `allow-passthrough` option from the window's options; if it is not set, the function returns 0.
    - Logs the input buffer content for debugging purposes.
    - If the input buffer length is greater than or equal to the length of the prefix 'tmux;', it checks if the input starts with this prefix.
    - If it does, it writes the raw string from the input buffer (excluding the prefix) to the screen context, considering passthrough options.
- **Output**:
    - The function returns 0, indicating successful processing of the input without any output value.
- **Functions called**:
    - [`input_split`](#input_split)
    - [`input_get`](#input_get)
    - [`sixel_parse`](image-sixel.c.driver.md#sixel_parse)
    - [`screen_write_sixelimage`](screen-write.c.driver.md#screen_write_sixelimage)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`screen_write_rawstring`](screen-write.c.driver.md#screen_write_rawstring)


---
### input_enter_osc<!-- {{#callable:input_enter_osc}} -->
The `input_enter_osc` function initializes the input context for processing OSC (Operating System Command) sequences.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
- **Control Flow**:
    - Logs the function entry using `log_debug`.
    - Calls [`input_clear`](#input_clear) to reset the input context state.
    - Calls [`input_start_timer`](#input_start_timer) to start a timer for the input context.
    - Clears the `INPUT_LAST` flag in the `flags` member of the input context.
- **Output**:
    - The function does not return a value; it modifies the state of the input context to prepare for OSC input processing.
- **Functions called**:
    - [`input_clear`](#input_clear)
    - [`input_start_timer`](#input_start_timer)


---
### input_exit_osc<!-- {{#callable:input_exit_osc}} -->
The `input_exit_osc` function processes the end of an Operating System Command (OSC) sequence, interpreting the command and executing the appropriate action.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for the input being processed, including the input buffer and associated window pane.
- **Control Flow**:
    - Checks if the `INPUT_DISCARD` flag is set in the `ictx` structure; if so, the function returns immediately.
    - Validates that the input length is greater than 0 and that the first character of the input buffer is a digit (0-9); if not, it returns.
    - Logs the current function name and the input buffer content for debugging purposes.
    - Parses the numeric option from the input buffer until a semicolon or null terminator is encountered.
    - If the next character is not a semicolon or null, it returns.
    - If the next character is a semicolon, it increments the pointer to skip it.
    - Uses a switch statement to handle different OSC options, calling specific functions or performing actions based on the parsed option.
- **Output**:
    - The function does not return a value but may modify the state of the `input_ctx`, update the window pane title, or perform other actions based on the parsed OSC command.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`screen_set_title`](screen.c.driver.md#screen_set_title)
    - [`notify_pane`](notify.c.driver.md#notify_pane)
    - [`server_redraw_window_borders`](server-fn.c.driver.md#server_redraw_window_borders)
    - [`server_status_window`](server-fn.c.driver.md#server_status_window)
    - [`input_osc_4`](#input_osc_4)
    - [`utf8_isvalid`](utf8.c.driver.md#utf8_isvalid)
    - [`screen_set_path`](screen.c.driver.md#screen_set_path)
    - [`input_osc_8`](#input_osc_8)
    - [`input_osc_10`](#input_osc_10)
    - [`input_osc_11`](#input_osc_11)
    - [`input_osc_12`](#input_osc_12)
    - [`input_osc_52`](#input_osc_52)
    - [`input_osc_104`](#input_osc_104)
    - [`input_osc_110`](#input_osc_110)
    - [`input_osc_111`](#input_osc_111)
    - [`input_osc_112`](#input_osc_112)
    - [`input_osc_133`](#input_osc_133)


---
### input_enter_apc<!-- {{#callable:input_enter_apc}} -->
The `input_enter_apc` function initializes the input context for an Application Programming Command (APC) sequence.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
- **Control Flow**:
    - Logs the function entry using `log_debug`.
    - Calls [`input_clear`](#input_clear) to reset the input context state.
    - Starts a timer for the input context using [`input_start_timer`](#input_start_timer).
    - Clears the `INPUT_LAST` flag from the `flags` member of the input context.
- **Output**:
    - The function does not return a value; it modifies the state of the input context to prepare for processing an APC sequence.
- **Functions called**:
    - [`input_clear`](#input_clear)
    - [`input_start_timer`](#input_start_timer)


---
### input_exit_apc<!-- {{#callable:input_exit_apc}} -->
The `input_exit_apc` function processes the end of an Application Protocol Command (APC) sequence, updating the terminal title if applicable.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input processing, including the input buffer and associated window pane.
- **Control Flow**:
    - Checks if the `INPUT_DISCARD` flag is set in the `ictx` structure; if so, the function returns immediately without processing.
    - Logs the current function name and the content of the input buffer for debugging purposes.
    - Calls [`screen_set_title`](screen.c.driver.md#screen_set_title) to set the terminal title using the content of the input buffer.
    - If the title is successfully set and the associated window pane (`wp`) is not NULL, it sends a notification to update the pane title and redraws the window borders and status.
- **Output**:
    - The function does not return a value; it modifies the terminal state and may trigger visual updates in the associated window pane.
- **Functions called**:
    - [`screen_set_title`](screen.c.driver.md#screen_set_title)
    - [`notify_pane`](notify.c.driver.md#notify_pane)
    - [`server_redraw_window_borders`](server-fn.c.driver.md#server_redraw_window_borders)
    - [`server_status_window`](server-fn.c.driver.md#server_status_window)


---
### input_enter_rename<!-- {{#callable:input_enter_rename}} -->
The `input_enter_rename` function initializes the input context for renaming operations.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
- **Control Flow**:
    - Logs the function name for debugging purposes.
    - Calls [`input_clear`](#input_clear) to reset the input context.
    - Starts a timer for the input context using [`input_start_timer`](#input_start_timer).
    - Clears the `INPUT_LAST` flag from the `flags` member of the input context.
- **Output**:
    - The function does not return a value; it modifies the state of the input context for renaming operations.
- **Functions called**:
    - [`input_clear`](#input_clear)
    - [`input_start_timer`](#input_start_timer)


---
### input_exit_rename<!-- {{#callable:input_exit_rename}} -->
The `input_exit_rename` function handles the termination of a rename operation for a window pane.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for the input operation, including the window pane and input buffer.
- **Control Flow**:
    - Checks if the `window_pane` pointer in `ictx` is NULL; if so, the function returns immediately.
    - Checks if the `INPUT_DISCARD` flag is set in `ictx`; if so, the function returns.
    - Checks if renaming is allowed by retrieving the `allow-rename` option; if not allowed, the function returns.
    - Logs the current input buffer for debugging purposes.
    - Validates the UTF-8 encoding of the input buffer; if invalid, the function returns.
    - If the input length is zero, it checks for the `automatic-rename` option and clears the window name if necessary.
    - If the input length is greater than zero, it sets the window name to the value in the input buffer and disables automatic renaming.
    - Calls functions to redraw the window borders and update the status of the window.
- **Output**:
    - The function does not return a value but modifies the name of the window pane based on the input buffer and updates the window's visual representation.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`utf8_isvalid`](utf8.c.driver.md#utf8_isvalid)
    - [`options_get_only`](options.c.driver.md#options_get_only)
    - [`options_remove_or_default`](options.c.driver.md#options_remove_or_default)
    - [`window_set_name`](window.c.driver.md#window_set_name)
    - [`options_set_number`](options.c.driver.md#options_set_number)
    - [`server_redraw_window_borders`](server-fn.c.driver.md#server_redraw_window_borders)
    - [`server_status_window`](server-fn.c.driver.md#server_status_window)


---
### input_top_bit_set<!-- {{#callable:input_top_bit_set}} -->
The `input_top_bit_set` function processes the top bit of input characters for UTF-8 encoding.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
- **Control Flow**:
    - Clears the `INPUT_LAST` flag from the `flags` field of the `input_ctx` structure.
    - Checks if UTF-8 processing has started; if not, attempts to open a new UTF-8 sequence with the current character.
    - If the UTF-8 sequence is already started, it appends the character to the existing sequence.
    - Handles the result of the UTF-8 appending: if more data is needed, it returns; if there's an error, it resets the UTF-8 state; if done, it processes the complete character.
    - Logs the details of the completed UTF-8 character.
    - Copies the UTF-8 data to the current cell and adds it to the screen write context.
    - Updates the last character processed and sets the `INPUT_LAST` flag.
- **Output**:
    - The function returns 0 upon successful processing of the input character, indicating that no errors occurred.
- **Functions called**:
    - [`utf8_open`](utf8.c.driver.md#utf8_open)
    - [`utf8_append`](utf8.c.driver.md#utf8_append)
    - [`utf8_copy`](utf8.c.driver.md#utf8_copy)
    - [`screen_write_collect_add`](screen-write.c.driver.md#screen_write_collect_add)


---
### input_osc_colour_reply<!-- {{#callable:input_osc_colour_reply}} -->
The `input_osc_colour_reply` function sends a color reply to the terminal based on the provided color value.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
    - `n`: An unsigned integer representing the color index or identifier.
    - `c`: An integer representing the color value, which may be transformed into RGB format.
- **Control Flow**:
    - If `c` is not -1, it is converted to RGB format using [`colour_force_rgb`](colour.c.driver.md#colour_force_rgb).
    - If the resulting `c` is -1, the function returns early without sending a reply.
    - The RGB components are extracted from `c` using [`colour_split_rgb`](colour.c.driver.md#colour_split_rgb).
    - The termination character for the reply is determined based on the `input_end` state of `ictx`.
    - The formatted reply string is constructed and sent to the terminal using [`input_reply`](#input_reply).
- **Output**:
    - The function does not return a value; instead, it sends a formatted color reply string to the terminal.
- **Functions called**:
    - [`colour_force_rgb`](colour.c.driver.md#colour_force_rgb)
    - [`colour_split_rgb`](colour.c.driver.md#colour_split_rgb)
    - [`input_reply`](#input_reply)


---
### input_osc_4<!-- {{#callable:input_osc_4}} -->
The `input_osc_4` function processes OSC 4 escape sequences to set or query color palette entries.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
    - `p`: A string containing the parameters for the OSC 4 command, which includes color indices and values.
- **Control Flow**:
    - The function duplicates the input string `p` to `copy` and initializes a pointer `s` to traverse it.
    - It enters a loop that continues until the end of the string is reached.
    - Within the loop, it parses an index from the string and checks for a semicolon delimiter.
    - If the index is valid (0 to 255), it checks if the next parameter is a query ('?') or a color value.
    - If it's a query, it retrieves the color from the palette and sends a reply.
    - If it's a color value, it attempts to parse it and set it in the palette.
    - If any parsing error occurs, it sets a flag to indicate a bad input.
    - After processing all parameters, it checks if a redraw is needed and performs it if necessary.
    - Finally, it frees the duplicated string.
- **Output**:
    - The function does not return a value but may modify the color palette and trigger a screen redraw if changes were made.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`strsep`](compat/strsep.c.driver.md#strsep)
    - [`colour_palette_get`](colour.c.driver.md#colour_palette_get)
    - [`input_osc_colour_reply`](#input_osc_colour_reply)
    - [`colour_parseX11`](colour.c.driver.md#colour_parseX11)
    - [`colour_palette_set`](colour.c.driver.md#colour_palette_set)
    - [`screen_write_fullredraw`](screen-write.c.driver.md#screen_write_fullredraw)


---
### input_osc_8<!-- {{#callable:input_osc_8}} -->
The `input_osc_8` function processes OSC 8 escape sequences to embed hyperlinks in the terminal.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
    - `p`: A string containing the parameters for the OSC 8 sequence, which may include an optional id and a URI.
- **Control Flow**:
    - The function initializes local variables for hyperlink context and starts parsing the input string `p`.
    - It uses a loop to find parameters separated by ':' or ';', checking for an 'id=' parameter and extracting the URI.
    - If the URI is empty, it sets the link to 0 and frees any allocated id.
    - If a valid URI is found, it stores the hyperlink in the context and logs the hyperlink information.
    - If any parsing errors occur, it jumps to the 'bad' label to handle the error.
- **Output**:
    - The function does not return a value but modifies the `grid_cell` structure to store the hyperlink information and logs debug messages.
- **Functions called**:
    - [`xstrndup`](xmalloc.c.driver.md#xstrndup)
    - [`hyperlinks_put`](hyperlinks.c.driver.md#hyperlinks_put)


---
### input_osc_10<!-- {{#callable:input_osc_10}} -->
The `input_osc_10` function handles the OSC 10 escape sequence for setting and querying the foreground color.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input handling.
    - `p`: A string representing the parameters passed with the OSC 10 sequence, which can be a color value or a query indicator ('?').
- **Control Flow**:
    - Checks if the input string `p` is a query ('?') to retrieve the current foreground color.
    - If `wp` (window pane) is NULL, the function returns early.
    - Retrieves the foreground color from the control client or defaults if not set.
    - If `p` is not a query, it attempts to parse the color value from `p` using [`colour_parseX11`](colour.c.driver.md#colour_parseX11).
    - If the color parsing fails, it logs an error and returns.
    - If a valid color is parsed and a palette exists, it updates the foreground color in the palette and marks the pane as changed.
- **Output**:
    - The function does not return a value but may send a reply to the terminal with the current foreground color if queried, or update the foreground color in the context's color palette.
- **Functions called**:
    - [`window_pane_get_fg_control_client`](window.c.driver.md#window_pane_get_fg_control_client)
    - [`tty_default_colours`](tty.c.driver.md#tty_default_colours)
    - [`window_pane_get_fg`](window.c.driver.md#window_pane_get_fg)
    - [`input_osc_colour_reply`](#input_osc_colour_reply)
    - [`colour_parseX11`](colour.c.driver.md#colour_parseX11)
    - [`screen_write_fullredraw`](screen-write.c.driver.md#screen_write_fullredraw)


---
### input_osc_110<!-- {{#callable:input_osc_110}} -->
The `input_osc_110` function resets the foreground color of the terminal to a default value.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input handling.
    - `p`: A string parameter that is expected to be empty for this function to execute its logic.
- **Control Flow**:
    - The function first checks if the string pointed to by `p` is not empty; if it is not, the function returns immediately without making any changes.
    - If the `palette` member of the `input_ctx` structure is not NULL, it sets the foreground color of the palette to 8 (a default color).
    - If the `window_pane` associated with the `input_ctx` is not NULL, it updates the pane's flags to indicate that its style has changed.
    - Finally, it calls [`screen_write_fullredraw`](screen-write.c.driver.md#screen_write_fullredraw) to trigger a full redraw of the screen context.
- **Output**:
    - The function does not return a value; it modifies the state of the input context and triggers a screen redraw.
- **Functions called**:
    - [`screen_write_fullredraw`](screen-write.c.driver.md#screen_write_fullredraw)


---
### input_osc_11<!-- {{#callable:input_osc_11}} -->
The `input_osc_11` function handles the OSC 11 escape sequence for setting or querying the background color.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input handling.
    - `p`: A string representing the parameters passed with the OSC 11 sequence, which can be a color value or a query indicator ('?').
- **Control Flow**:
    - The function first checks if the input string `p` is equal to '?'. If it is, it retrieves the current background color from the associated window pane and sends a reply with that color.
    - If `p` is not '?', it attempts to parse the color value from the string using [`colour_parseX11`](colour.c.driver.md#colour_parseX11). If parsing fails, it logs an error and exits.
    - If the color is successfully parsed and the palette is not NULL, it updates the background color in the palette and marks the window pane as changed, triggering a full redraw of the screen.
- **Output**:
    - The function does not return a value but sends a reply with the current background color if queried, and updates the background color in the context's palette if a valid color is provided.
- **Functions called**:
    - [`window_pane_get_bg`](window.c.driver.md#window_pane_get_bg)
    - [`input_osc_colour_reply`](#input_osc_colour_reply)
    - [`colour_parseX11`](colour.c.driver.md#colour_parseX11)
    - [`screen_write_fullredraw`](screen-write.c.driver.md#screen_write_fullredraw)


---
### input_osc_111<!-- {{#callable:input_osc_111}} -->
The `input_osc_111` function resets the background color of the terminal to a default value.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
    - `p`: A string parameter that is expected to be empty for this function to execute its logic.
- **Control Flow**:
    - The function first checks if the input string `p` is not empty; if it is not, the function returns immediately without making any changes.
    - If the `palette` member of the `input_ctx` structure is not NULL, it sets the background color of the palette to 8.
    - If the `window_pane` associated with the `input_ctx` is not NULL, it updates the pane's flags to indicate that the style and theme have changed.
    - Finally, it triggers a full redraw of the screen using the [`screen_write_fullredraw`](screen-write.c.driver.md#screen_write_fullredraw) function.
- **Output**:
    - The function does not return a value but modifies the state of the terminal's background color and triggers a redraw if applicable.
- **Functions called**:
    - [`screen_write_fullredraw`](screen-write.c.driver.md#screen_write_fullredraw)


---
### input_osc_12<!-- {{#callable:input_osc_12}} -->
The `input_osc_12` function handles the OSC 12 escape sequence for setting and querying the cursor color.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input handling.
    - `p`: A string representing the parameters passed with the OSC 12 sequence, which can be a color value or a query indicator ('?').
- **Control Flow**:
    - The function first checks if the input string `p` is equal to '?'.
    - If it is, and if the `window_pane` associated with `ictx` is not NULL, it retrieves the current cursor color.
    - If the cursor color is -1, it falls back to the default cursor color.
    - It then calls [`input_osc_colour_reply`](#input_osc_colour_reply) to send the current cursor color back to the terminal.
    - If `p` is not '?', it attempts to parse the color value from `p` using [`colour_parseX11`](colour.c.driver.md#colour_parseX11).
    - If the parsing fails (returns -1), it logs a debug message and exits.
    - If successful, it sets the cursor color using [`screen_set_cursor_colour`](screen.c.driver.md#screen_set_cursor_colour).
- **Output**:
    - The function does not return a value but modifies the state of the terminal by setting the cursor color or replying with the current cursor color.
- **Functions called**:
    - [`input_osc_colour_reply`](#input_osc_colour_reply)
    - [`colour_parseX11`](colour.c.driver.md#colour_parseX11)
    - [`screen_set_cursor_colour`](screen.c.driver.md#screen_set_cursor_colour)


---
### input_osc_112<!-- {{#callable:input_osc_112}} -->
The `input_osc_112` function resets the cursor color to the default when no arguments are provided.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
    - `p`: A pointer to a string that represents the parameters for the OSC (Operating System Command) sequence.
- **Control Flow**:
    - The function checks if the first character of the string `p` is the null terminator, indicating no arguments were provided.
    - If no arguments are allowed, it calls [`screen_set_cursor_colour`](screen.c.driver.md#screen_set_cursor_colour) with the context's screen and a value of -1 to reset the cursor color.
- **Output**:
    - The function does not return a value; it modifies the cursor color on the screen based on the provided parameters.
- **Functions called**:
    - [`screen_set_cursor_colour`](screen.c.driver.md#screen_set_cursor_colour)


---
### input_osc_133<!-- {{#callable:input_osc_133}} -->
The `input_osc_133` function processes OSC 133 escape sequences to set flags on a specific grid line.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input processing.
    - `p`: A pointer to a string containing the parameters for the OSC 133 command.
- **Control Flow**:
    - The function retrieves the current grid from the `input_ctx` structure.
    - It calculates the current line based on the cursor's vertical position.
    - If the calculated line exceeds the grid's height, the function returns early.
    - The function retrieves the grid line at the calculated position.
    - It checks the first character of the input string to determine the action: if it's 'A', it sets the `GRID_LINE_START_PROMPT` flag; if it's 'C', it sets the `GRID_LINE_START_OUTPUT` flag.
- **Output**:
    - The function does not return a value; it modifies the flags of the specified grid line based on the input parameters.
- **Functions called**:
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)


---
### input_osc_52<!-- {{#callable:input_osc_52}} -->
The `input_osc_52` function handles the OSC 52 escape sequence for setting the clipboard content.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input handling.
    - `p`: A string containing the parameters for the OSC 52 command.
- **Control Flow**:
    - Checks if the `window_pane` associated with the input context is NULL and returns if it is.
    - Retrieves the clipboard setting state and returns if it is not set to allow clipboard operations.
    - Finds the first semicolon in the input string to separate the flags from the clipboard data.
    - Logs the debug information about the input string and flags.
    - If the input string after the semicolon is a question mark, retrieves the current clipboard content and sends it back.
    - If the input string contains data, decodes it from base64 and sets the selection in the screen context.
    - Notifies the pane that the clipboard has been set and adds the new clipboard content to the paste buffer.
- **Output**:
    - The function does not return a value but modifies the clipboard content and updates the screen context accordingly.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`paste_get_top`](paste.c.driver.md#paste_get_top)
    - [`paste_buffer_data`](paste.c.driver.md#paste_buffer_data)
    - [`input_reply_clipboard`](#input_reply_clipboard)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`b64_pton`](compat/base64.c.driver.md#b64_pton)
    - [`screen_write_start_pane`](screen-write.c.driver.md#screen_write_start_pane)
    - [`screen_write_setselection`](screen-write.c.driver.md#screen_write_setselection)
    - [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop)
    - [`notify_pane`](notify.c.driver.md#notify_pane)
    - [`paste_add`](paste.c.driver.md#paste_add)


---
### input_osc_104<!-- {{#callable:input_osc_104}} -->
The `input_osc_104` function processes OSC 104 sequences to unset specified color palette entries.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
    - `p`: A string containing the parameters for the OSC 104 sequence, which specifies the indices of the color palette entries to unset.
- **Control Flow**:
    - If the input string `p` is empty, the function clears the color palette and triggers a full redraw of the screen.
    - The function duplicates the input string `p` for processing and iterates through it to extract indices.
    - For each index extracted, it checks if it is valid (between 0 and 255) and attempts to unset the corresponding color in the palette.
    - If any index is invalid, a debug message is logged.
    - If any palette entry is successfully unset, a flag is set to indicate that a redraw is needed.
    - Finally, if any changes were made, a full redraw of the screen is triggered.
- **Output**:
    - The function does not return a value but modifies the state of the color palette and may trigger a screen redraw.
- **Functions called**:
    - [`colour_palette_clear`](colour.c.driver.md#colour_palette_clear)
    - [`screen_write_fullredraw`](screen-write.c.driver.md#screen_write_fullredraw)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`colour_palette_set`](colour.c.driver.md#colour_palette_set)


---
### input_reply_clipboard<!-- {{#callable:input_reply_clipboard}} -->
The `input_reply_clipboard` function encodes a given buffer into base64 and sends it to a terminal as a clipboard update.
- **Inputs**:
    - `bev`: A pointer to a `bufferevent` structure representing the event buffer to which the clipboard data will be written.
    - `buf`: A pointer to a character array containing the data to be encoded and sent to the clipboard.
    - `len`: The length of the data in `buf` that needs to be encoded.
    - `end`: A pointer to a character array that indicates the end of the clipboard command.
- **Control Flow**:
    - Check if `buf` is not NULL and `len` is not zero.
    - If the length of the buffer exceeds a certain threshold, return early to avoid overflow.
    - Calculate the required length for the base64 output and allocate memory for it.
    - Encode the input buffer `buf` into base64 format using [`b64_ntop`](compat/base64.c.driver.md#b64_ntop).
    - If the encoding fails, free the allocated memory and return.
    - Write the clipboard command prefix to the `bufferevent`.
    - If the encoded output length is not zero, write the encoded data to the `bufferevent`.
    - Finally, write the end command to the `bufferevent` and free the allocated memory.
- **Output**:
    - The function does not return a value; instead, it sends the encoded clipboard data to the specified `bufferevent`.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`b64_ntop`](compat/base64.c.driver.md#b64_ntop)


---
### input_set_buffer_size<!-- {{#callable:input_set_buffer_size}} -->
Sets the input buffer size for the application.
- **Inputs**:
    - `buffer_size`: The new size for the input buffer, specified as a `size_t` value.
- **Control Flow**:
    - Logs the current function name and the previous and new buffer sizes using `log_debug`.
    - Updates the global variable `input_buffer_size` with the new buffer size provided as an argument.
- **Output**:
    - The function does not return a value; it modifies the global `input_buffer_size` variable.


---
### input_report_current_theme<!-- {{#callable:input_report_current_theme}} -->
Reports the current theme of the window pane.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input handling, including the current window pane.
- **Control Flow**:
    - The function retrieves the current theme of the window pane using [`window_pane_get_theme`](window.c.driver.md#window_pane_get_theme).
    - It then checks the theme value and responds accordingly: if the theme is `THEME_DARK`, it sends a response indicating this theme; if it is `THEME_LIGHT`, it sends a different response; if the theme is `THEME_UNKNOWN`, it does nothing.
- **Output**:
    - The function does not return a value but sends a response to the input context's event based on the current theme of the window pane.
- **Functions called**:
    - [`window_pane_get_theme`](window.c.driver.md#window_pane_get_theme)
    - [`input_reply`](#input_reply)


