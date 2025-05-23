# Purpose
The provided C source code is part of the `tmux` terminal multiplexer, specifically handling the parsing and processing of terminal input sequences. This file is responsible for interpreting various control sequences, escape sequences, and other input data that a terminal might receive. It includes functionality for handling ANSI and DEC terminal sequences, such as cursor movements, screen clearing, and color settings. The code is structured around a state machine that transitions between different states based on the input received, allowing it to handle complex sequences efficiently.

Key components of the code include structures for managing input context (`input_ctx`), input parameters (`input_param`), and input states (`input_state`). The code defines numerous functions for handling specific sequences, such as [`input_csi_dispatch`](#input_csi_dispatch) for CSI sequences and [`input_osc_4`](#input_osc_4) for OSC sequences. It also includes mechanisms for managing terminal attributes like colors and cursor styles, and it supports features like setting the clipboard and handling hyperlinks. The file is designed to be integrated into the larger `tmux` application, providing essential functionality for interpreting and responding to terminal input in a way that supports the multiplexing of terminal sessions.
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
    - `cell`: A nested structure of type `grid_cell` representing the cell's properties.
    - `set`: An integer indicating if the cell is set.
    - `g0set`: An integer flag indicating if the G0 character set is active (1 if ACS).
    - `g1set`: An integer flag indicating if the G1 character set is active (1 if ACS).
- **Description**: The `input_cell` structure is used to represent a cell in an input parser, specifically for handling terminal input sequences. It contains a `grid_cell` structure to hold the cell's properties, and several integer flags to indicate the active character set and whether the cell is set. This structure is part of a larger system for parsing and handling terminal input, allowing for the management of character sets and cell states within a grid or screen.


---
### input_param
- **Type**: `struct`
- **Members**:
    - `type`: An enum indicating the type of input parameter, which can be missing, a number, or a string.
    - `num`: An integer value used when the input parameter is a number.
    - `str`: A pointer to a character string used when the input parameter is a string.
- **Description**: The `input_param` structure is used to represent an input parameter in a parser, which can be either a number or a string. It uses an enum to specify the type of the parameter and a union to store the actual value, either as an integer or a string pointer. This allows the parser to handle different types of input parameters flexibly.


---
### input_ctx
- **Type**: `struct`
- **Members**:
    - `wp`: Pointer to the associated window pane.
    - `event`: Pointer to the bufferevent structure for handling I/O events.
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
    - `param_list_len`: Number of parameters in the parameter list.
    - `utf8data`: Data structure for handling UTF-8 characters.
    - `utf8started`: Flag indicating if UTF-8 processing has started.
    - `ch`: Current character being processed.
    - `last`: Last processed UTF-8 data.
    - `flags`: Flags for input processing state.
    - `state`: Pointer to the current input state.
    - `timer`: Event timer for managing input timeouts.
    - `since_ground`: Buffer for storing input since the last ground state.
- **Description**: The `input_ctx` structure is a comprehensive context for managing input parsing and processing in a terminal emulator. It maintains the state of the input parser, including the current window pane, event handling, screen writing context, and color palette. It also tracks the current and previous input cells, cursor positions, and screen modes. The structure includes buffers for handling intermediate characters, parameters, and input data, as well as flags and state pointers for managing the input parsing state machine. Additionally, it supports UTF-8 character processing and manages input timeouts with an event timer.


---
### input_table_entry
- **Type**: `struct`
- **Members**:
    - `ch`: An integer representing a character code.
    - `interm`: A pointer to a constant character string, used as an intermediate string.
    - `type`: An integer representing the type of input command.
- **Description**: The `input_table_entry` structure is used to define entries in a command table for parsing input sequences. Each entry consists of a character code (`ch`), an intermediate string (`interm`), and a type identifier (`type`). This structure is part of a larger input parsing system, likely used to handle escape sequences or control sequences in a terminal emulator or similar application.


---
### input_esc_type
- **Type**: `enum`
- **Members**:
    - `INPUT_ESC_DECALN`: Represents the escape sequence for DEC alignment test.
    - `INPUT_ESC_DECKPAM`: Represents the escape sequence for enabling the keypad application mode.
    - `INPUT_ESC_DECKPNM`: Represents the escape sequence for disabling the keypad application mode.
    - `INPUT_ESC_DECRC`: Represents the escape sequence for restoring the cursor position.
    - `INPUT_ESC_DECSC`: Represents the escape sequence for saving the cursor position.
    - `INPUT_ESC_HTS`: Represents the escape sequence for setting a horizontal tab stop.
    - `INPUT_ESC_IND`: Represents the escape sequence for index (move cursor down one line).
    - `INPUT_ESC_NEL`: Represents the escape sequence for next line (move cursor to the beginning of the next line).
    - `INPUT_ESC_RI`: Represents the escape sequence for reverse index (move cursor up one line).
    - `INPUT_ESC_RIS`: Represents the escape sequence for reset to initial state.
    - `INPUT_ESC_SCSG0_OFF`: Represents the escape sequence for setting G0 character set to standard.
    - `INPUT_ESC_SCSG0_ON`: Represents the escape sequence for setting G0 character set to alternate.
    - `INPUT_ESC_SCSG1_OFF`: Represents the escape sequence for setting G1 character set to standard.
    - `INPUT_ESC_SCSG1_ON`: Represents the escape sequence for setting G1 character set to alternate.
    - `INPUT_ESC_ST`: Represents the escape sequence for string terminator.
- **Description**: The `input_esc_type` is an enumeration that defines various escape sequences used in terminal emulation. Each enumerator corresponds to a specific escape sequence that performs a particular function, such as moving the cursor, setting character sets, or resetting the terminal state. These escape sequences are part of the control sequences used to manipulate the terminal display and behavior.


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
- **Description**: The `input_csi_type` is an enumeration that defines a set of constants representing various Control Sequence Introducer (CSI) commands used in terminal emulation. Each enumerator corresponds to a specific CSI command, which is used to control cursor movement, screen manipulation, and other terminal functions. This enumeration is part of a larger input parsing system that interprets and executes terminal control sequences.


---
### input_transition
- **Type**: `struct`
- **Members**:
    - `first`: An integer representing the first character in a range for a transition.
    - `last`: An integer representing the last character in a range for a transition.
    - `handler`: A pointer to a function that handles the transition, taking a pointer to an input context as an argument.
    - `state`: A pointer to a constant input_state structure representing the state to transition to.
- **Description**: The `input_transition` structure defines a state transition in an input parser, specifying a range of characters (`first` to `last`) that trigger the transition, a handler function to execute during the transition, and the next state to transition to. This structure is used in a state machine to manage input parsing, allowing for flexible handling of different input sequences.


---
### input_state
- **Type**: `struct`
- **Members**:
    - `name`: A constant character pointer representing the name of the input state.
    - `enter`: A function pointer to a function that is called when entering the input state.
    - `exit`: A function pointer to a function that is called when exiting the input state.
    - `transitions`: A constant pointer to an array of input_transition structures defining state transitions.
- **Description**: The `input_state` structure defines a state in an input parsing state machine, typically used in terminal emulation or similar applications. It includes a name for the state, function pointers for actions to perform when entering or exiting the state, and a list of transitions to other states. This structure is part of a larger system that processes input sequences and changes states based on defined transitions, allowing for complex input handling and parsing.


# Functions

---
### printflike<!-- {{#callable:printflike}} -->
The `printflike` function formats and sends a reply to the terminal based on a variable argument list.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for the input being processed.
    - `fmt`: A format string that specifies how to format the output.
    - `...`: A variable number of arguments that are used to format the output according to the format string.
- **Control Flow**:
    - The function checks if the `event` field of the `input_ctx` structure is NULL; if it is, the function returns immediately.
    - The function initializes a variable argument list using `va_start` to process the additional arguments.
    - It uses `xvasprintf` to format the output string based on the provided format and arguments.
    - The formatted string is then sent to the terminal using `bufferevent_write`.
    - Finally, the function frees the allocated memory for the formatted string.
- **Output**:
    - The function does not return a value; instead, it sends a formatted string to the terminal as a reply.


---
### input_timer_callback<!-- {{#callable:input_timer_callback}} -->
The `input_timer_callback` function handles the expiration of an input timer by logging the current state and resetting the input context.
- **Inputs**:
    - `fd`: An unused file descriptor, typically for event handling.
    - `events`: An unused short representing event types.
    - `arg`: A pointer to the `input_ctx` structure containing the input context.
- **Control Flow**:
    - The function begins by casting the `arg` parameter to a pointer of type `input_ctx`.
    - It logs a debug message indicating that the timer has expired, including the name of the current state.
    - Finally, it calls the [`input_reset`](#input_reset) function to reset the input context.
- **Output**:
    - The function does not return a value; it performs actions that affect the state of the input context and logs a message.
- **Functions called**:
    - [`input_reset`](#input_reset)


---
### input_start_timer<!-- {{#callable:input_start_timer}} -->
Starts a timer for the input context.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input handling.
- **Control Flow**:
    - Initializes a `struct timeval` with a duration of 5 seconds.
    - Calls `event_del` to remove any existing timer associated with the input context.
    - Calls `event_add` to add a new timer event with the specified duration.
- **Output**:
    - The function does not return a value; it modifies the state of the timer associated with the input context.


---
### input_reset_cell<!-- {{#callable:input_reset_cell}} -->
Resets the input cell state to default values.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
- **Control Flow**:
    - The function begins by copying the default cell attributes from `grid_default_cell` to the current cell in the `input_ctx` structure.
    - It then resets the `set`, `g0set`, and `g1set` flags of the current cell to 0.
    - Next, it copies the current cell's state to `old_cell` to preserve the previous state.
    - Finally, it resets the old cursor positions (`old_cx` and `old_cy`) to 0.
- **Output**:
    - The function does not return a value; it modifies the state of the `input_ctx` structure directly.


---
### input_save_state<!-- {{#callable:input_save_state}} -->
The `input_save_state` function saves the current state of the input context.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
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
Restores the previous input state, including cursor position and cell attributes.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
- **Control Flow**:
    - Copies the current cell state from `old_cell` to `cell` using `memcpy`.
    - Checks if the `old_mode` has the `MODE_ORIGIN` flag set.
    - If `MODE_ORIGIN` is set, it calls `screen_write_mode_set` to restore the origin mode.
    - If `MODE_ORIGIN` is not set, it calls `screen_write_mode_clear` to clear the origin mode.
    - Moves the cursor to the previous position using `screen_write_cursormove`.
- **Output**:
    - The function does not return a value; it modifies the state of the input context and the screen directly.


---
### input_init<!-- {{#callable:input_init}} -->
Initializes an `input_ctx` structure for handling input in a terminal pane.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure representing the pane associated with the input context.
    - `bev`: A pointer to a `bufferevent` structure used for handling buffered events.
    - `palette`: A pointer to a `colour_palette` structure for managing color settings.
- **Control Flow**:
    - Allocates memory for a new `input_ctx` structure using `xcalloc`.
    - Assigns the provided `window_pane`, `bufferevent`, and `colour_palette` to the corresponding fields in the `input_ctx` structure.
    - Initializes the input buffer size and allocates memory for the input buffer.
    - Creates a new event buffer for tracking input received since the last ground state.
    - Sets up a timer for handling input timeouts using `evtimer_set`.
    - Calls [`input_reset`](#input_reset) to initialize the input context to a default state.
- **Output**:
    - Returns a pointer to the initialized `input_ctx` structure.
- **Functions called**:
    - [`input_reset`](#input_reset)


---
### input_free<!-- {{#callable:input_free}} -->
The `input_free` function deallocates memory associated with an `input_ctx` structure.
- **Inputs**:
    - `ictx`: A pointer to an `input_ctx` structure that holds the context for input parsing.
- **Control Flow**:
    - Iterates over the `param_list` of the `input_ctx` to free any allocated strings of type `INPUT_STRING`.
    - Calls `event_del` to remove the associated timer event.
    - Frees the `input_buf` buffer allocated for input data.
    - Frees the `since_ground` buffer that holds input received since the last ground state.
    - Finally, frees the `input_ctx` structure itself.
- **Output**:
    - The function does not return a value; it performs cleanup and deallocates resources associated with the `input_ctx`.


---
### input_reset<!-- {{#callable:input_reset}} -->
Resets the input context and optionally clears the screen.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
    - `clear`: An integer flag indicating whether to clear the screen (non-zero value) or not (zero value).
- **Control Flow**:
    - Calls [`input_reset_cell`](#input_reset_cell) to reset the input cell state.
    - If `clear` is true and the associated `window_pane` (`wp`) is not NULL, it checks if the `modes` list is empty.
    - If the `modes` list is empty, it starts writing to the pane using `screen_write_start_pane`, otherwise it uses `screen_write_start`.
    - Resets the screen context with `screen_write_reset` and stops writing with `screen_write_stop`.
    - Calls [`input_clear`](#input_clear) to clear the input context.
    - Sets the `state` of the input context to `input_state_ground` and resets the `flags` to zero.
- **Output**:
    - The function does not return a value; it modifies the state of the input context and the screen based on the provided parameters.
- **Functions called**:
    - [`input_reset_cell`](#input_reset_cell)
    - [`input_clear`](#input_clear)


---
### input_pending<!-- {{#callable:input_pending}} -->
Returns the `evbuffer` containing all input received since the last ground state.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context of the input parser.
- **Control Flow**:
    - The function directly accesses the `since_ground` member of the `input_ctx` structure.
    - It returns the `evbuffer` associated with the `since_ground` member.
- **Output**:
    - Returns a pointer to an `evbuffer` that contains all input received since the last ground state.


---
### input_set_state<!-- {{#callable:input_set_state}} -->
Sets the input context's state based on a transition.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure representing the current input context.
    - `itr`: A pointer to the `input_transition` structure that contains the new state and associated handlers.
- **Control Flow**:
    - Checks if the current state's exit handler is not NULL and calls it if present.
    - Updates the current state of the input context to the new state specified in the transition.
    - Checks if the new state's enter handler is not NULL and calls it if present.
- **Output**:
    - The function does not return a value; it modifies the state of the input context directly.


---
### input_parse<!-- {{#callable:input_parse}} -->
Parses input data from a buffer and manages state transitions based on the input.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input parsing.
    - `buf`: A pointer to a buffer containing the input data to be parsed.
    - `len`: The length of the input data in the buffer.
- **Control Flow**:
    - The function initializes a loop that continues until all input data has been processed.
    - For each character in the input buffer, it checks if a state transition is needed based on the current state and the character.
    - If a transition is required, it searches through the available transitions for the current state to find a matching handler.
    - If a handler is found, it is executed, and the state may be updated based on the transition.
    - If the handler does not return a non-zero value, the state is updated to the new state defined by the transition.
    - If the current state is not the ground state, the character is added to the `since_ground` buffer.
- **Output**:
    - The function does not return a value but modifies the state of the `input_ctx` and may update the `since_ground` buffer with processed input.
- **Functions called**:
    - [`input_set_state`](#input_set_state)


---
### input_parse_pane<!-- {{#callable:input_parse_pane}} -->
The `input_parse_pane` function processes new input data for a specified window pane.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` that represents the pane from which input data is being parsed.
- **Control Flow**:
    - Calls `window_pane_get_new_data` to retrieve new input data and its size from the specified pane.
    - Invokes [`input_parse_buffer`](#input_parse_buffer) to process the retrieved input data.
    - Updates the pane's offset using `window_pane_update_used_data` to reflect the amount of data that has been processed.
- **Output**:
    - The function does not return a value; it modifies the state of the window pane based on the processed input data.
- **Functions called**:
    - [`input_parse_buffer`](#input_parse_buffer)


---
### input_parse_buffer<!-- {{#callable:input_parse_buffer}} -->
Parses input data from a buffer for a specific window pane.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane that is receiving input.
    - `buf`: A pointer to a buffer containing the input data to be parsed.
    - `len`: The length of the input data in the buffer.
- **Control Flow**:
    - If the length of the input data is zero, the function returns immediately.
    - The function updates the activity status of the window associated with the pane.
    - It sets the pane's flags to indicate that it has changed.
    - If the pane has modes set, it flags the pane as having unseen changes.
    - Depending on whether there are modes set, it starts the screen writing context for the pane or the base.
    - The function logs the input data received for debugging purposes.
    - It calls the [`input_parse`](#input_parse) function to handle the actual parsing of the input data.
    - Finally, it stops the screen writing context.
- **Output**:
    - The function does not return a value; it modifies the state of the window pane and processes the input data.
- **Functions called**:
    - [`input_parse`](#input_parse)


---
### input_parse_screen<!-- {{#callable:input_parse_screen}} -->
Parses input data for a screen context and processes it accordingly.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the input context.
    - `s`: A pointer to the `screen` structure representing the screen to which the input is directed.
    - `cb`: A callback function of type `screen_write_init_ctx_cb` used to initialize the screen write context.
    - `arg`: A pointer to user-defined data passed to the callback function.
    - `buf`: A pointer to a buffer containing the input data to be parsed.
    - `len`: The length of the input data in the buffer.
- **Control Flow**:
    - Checks if the length of the input data is zero; if so, it returns immediately.
    - Calls `screen_write_start_callback` to initialize the screen write context with the provided screen and callback.
    - Calls [`input_parse`](#input_parse) to process the input data from the buffer.
    - Finally, calls `screen_write_stop` to finalize the screen write context.
- **Output**:
    - The function does not return a value; it modifies the state of the screen and processes the input data.
- **Functions called**:
    - [`input_parse`](#input_parse)


---
### input_split<!-- {{#callable:input_split}} -->
The `input_split` function parses a parameter buffer into a list of input parameters.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input parsing, including the parameter buffer and the list of parsed parameters.
- **Control Flow**:
    - The function starts by freeing any previously allocated strings in the `param_list` of the `input_ctx` structure.
    - It resets the `param_list_len` to zero.
    - If the `param_len` is zero, it returns 0 immediately, indicating no parameters to process.
    - It enters a loop where it splits the `param_buf` string by semicolons, processing each segment.
    - For each segment, it checks if it is empty, assigns it as `INPUT_MISSING`, or determines if it is a string or number.
    - If it is a string, it duplicates the string; if it is a number, it converts it and checks for errors.
    - The parsed parameters are stored in the `param_list` of the `input_ctx` structure.
    - After parsing, it logs the type and value of each parameter.
    - Finally, it returns 0 to indicate successful parsing.
- **Output**:
    - The function returns 0 on success, or -1 if an error occurs during parsing, such as exceeding the maximum number of parameters or encountering an invalid number.


---
### input_get<!-- {{#callable:input_get}} -->
The `input_get` function retrieves a parameter value from the input context, ensuring it meets specified constraints.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input parameters.
    - `validx`: An unsigned integer representing the index of the parameter to retrieve.
    - `minval`: An integer specifying the minimum acceptable value for the parameter.
    - `defval`: An integer that serves as the default value to return if the parameter is invalid.
- **Control Flow**:
    - Checks if `validx` is greater than or equal to the length of the parameter list in `ictx`; if so, returns `defval`.
    - Retrieves the parameter at the specified index and checks its type.
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
    - `ap`: A variable argument list that contains the values to format into the reply message.
- **Control Flow**:
    - Check if the `event` member of `ictx` is NULL; if it is, the function returns immediately.
    - Initialize the variable argument list using `va_start` with `fmt`.
    - Call `xvasprintf` to format the reply message into a dynamically allocated string.
    - Log the formatted reply message for debugging purposes.
    - Send the formatted reply message to the terminal using `bufferevent_write`.
    - Free the dynamically allocated reply string.
- **Output**:
    - The function does not return a value; it sends a formatted reply message to the terminal.


---
### input_clear<!-- {{#callable:input_clear}} -->
The `input_clear` function resets the input context by clearing buffers and flags.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the state and buffers for input processing.
- **Control Flow**:
    - The function begins by deleting any active timer associated with the input context.
    - It then clears the intermediate buffer by setting its first character to null and resetting its length.
    - Next, it clears the parameter buffer in a similar manner.
    - The input buffer is also cleared, and its length is reset.
    - The input end state is set to `INPUT_END_ST`.
    - Finally, the `INPUT_DISCARD` flag is cleared from the context's flags.
- **Output**:
    - The function does not return a value; it modifies the state of the `input_ctx` structure directly by clearing its buffers and resetting flags.


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


---
### input_print<!-- {{#callable:input_print}} -->
The `input_print` function processes a character input and updates the screen context accordingly.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context, including the character to be printed and the screen write context.
- **Control Flow**:
    - The function initializes the `utf8started` flag to 0, indicating that the current character cannot be valid UTF-8.
    - It determines the character set to use based on the `set` value of the `cell` in the `input_ctx` structure.
    - If the character set is set to 1, it updates the cell's attributes to include `GRID_ATTR_CHARSET`, otherwise it clears this attribute.
    - The character is then set in the `cell.data` using the `utf8_set` function.
    - The character is added to the screen write context using `screen_write_collect_add`.
    - The last character processed is stored in `last` for potential future use.
    - The `INPUT_LAST` flag is set in the `flags` of the `input_ctx` to indicate that this was the last input character processed.
    - Finally, the `GRID_ATTR_CHARSET` attribute is cleared from the cell.
- **Output**:
    - The function returns 0, indicating successful processing of the input character.


---
### input_intermediate<!-- {{#callable:input_intermediate}} -->
The `input_intermediate` function collects intermediate input characters into a buffer.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
- **Control Flow**:
    - Checks if the length of the intermediate buffer has reached its maximum capacity minus one.
    - If the buffer is full, it sets the `INPUT_DISCARD` flag in the context's flags.
    - If there is space in the buffer, it adds the current character (`ch`) to the buffer and increments the buffer length.
    - Finally, it null-terminates the string in the buffer.
- **Output**:
    - The function returns 0, indicating successful execution.


---
### input_parameter<!-- {{#callable:input_parameter}} -->
The `input_parameter` function collects a character input into a parameter buffer within a given input context.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the state and data for input processing.
- **Control Flow**:
    - Checks if the current length of the parameter buffer (`param_len`) has reached its maximum capacity (size of `param_buf` minus one).
    - If the buffer is full, it sets the `INPUT_DISCARD` flag in the `flags` member of the `input_ctx` structure.
    - If there is space in the buffer, it appends the current character (`ch`) to the `param_buf`, increments the `param_len`, and null-terminates the string.
- **Output**:
    - The function returns 0, indicating successful execution without any errors.


---
### input_input<!-- {{#callable:input_input}} -->
The `input_input` function appends a character to the input buffer, dynamically resizing the buffer if necessary.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing, including the input buffer and its properties.
- **Control Flow**:
    - The function retrieves the current size of the input buffer from `ictx->input_space`.
    - It enters a loop that checks if the current length of the input plus one (for the new character) exceeds the available space.
    - If the space is insufficient, it doubles the size of the buffer until it is large enough or exceeds a predefined maximum size.
    - If the new size exceeds `input_buffer_size`, it sets a discard flag in `ictx->flags` and exits the function.
    - If there is enough space, it appends the character from `ictx->ch` to the input buffer and increments the input length.
    - Finally, it null-terminates the string in the input buffer.
- **Output**:
    - The function returns 0 upon successful execution, indicating that the character has been added to the input buffer.


---
### input_c0_dispatch<!-- {{#callable:input_c0_dispatch}} -->
The `input_c0_dispatch` function processes C0 control characters in the input context.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context, including the character to be processed and the associated screen context.
- **Control Flow**:
    - The function begins by resetting the UTF-8 state to invalid.
    - It logs the character being processed for debugging purposes.
    - A switch statement is used to handle different C0 control characters.
    - For the NUL character, no action is taken.
    - For the BEL character, an alert is queued if a window pane is associated.
    - For the BS (Backspace) character, the backspace action is performed on the screen.
    - For the HT (Horizontal Tab) character, the function calculates the next tab stop and updates the cursor position or sets a tab if the space is empty.
    - For LF (Line Feed), VT (Vertical Tab), and FF (Form Feed), a line feed is performed, and a carriage return is added if in CRLF mode.
    - For CR (Carriage Return), a carriage return is performed.
    - For SO (Shift Out) and SI (Shift In), the character set state is toggled.
    - If an unknown character is encountered, it is logged as such.
    - Finally, the INPUT_LAST flag is cleared before returning.
- **Output**:
    - The function returns 0, indicating successful processing of the input character.


---
### input_esc_dispatch<!-- {{#callable:input_esc_dispatch}} -->
The `input_esc_dispatch` function processes escape sequences in the input context.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context, including the state of the screen and input parameters.
- **Control Flow**:
    - Checks if the `INPUT_DISCARD` flag is set in the `ictx` structure; if so, it returns immediately.
    - Logs the current character and intermediate buffer for debugging purposes.
    - Performs a binary search on the `input_esc_table` to find a matching escape sequence entry based on the current character.
    - If no matching entry is found, it logs an unknown character message and returns.
    - Based on the type of the matched entry, it executes specific actions such as resetting the screen, handling line feeds, or modifying the cursor state.
    - Clears the `INPUT_LAST` flag from the `ictx` structure before returning.
- **Output**:
    - Returns 0 to indicate successful processing of the escape sequence, or does not return if the input is discarded.
- **Functions called**:
    - [`input_reset_cell`](#input_reset_cell)
    - [`input_save_state`](#input_save_state)
    - [`input_restore_state`](#input_restore_state)


---
### input_csi_dispatch<!-- {{#callable:input_csi_dispatch}} -->
The `input_csi_dispatch` function processes Control Sequence Introducer (CSI) commands for terminal input.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context, including the state of the terminal and the input buffer.
- **Control Flow**:
    - Checks if the `INPUT_DISCARD` flag is set in `ictx` and returns early if true.
    - Logs the current character and the contents of the intermediate and parameter buffers.
    - Calls [`input_split`](#input_split) to parse the parameter buffer; returns if it fails.
    - Uses binary search to find the corresponding entry in the `input_csi_table` based on the current character.
    - If no entry is found, logs an unknown character and returns.
    - Processes the command based on the type of the entry found, executing different actions such as moving the cursor, modifying screen attributes, or sending replies based on the command type.
- **Output**:
    - Returns 0 upon successful processing of the CSI command, or early returns in case of errors or unrecognized commands.
- **Functions called**:
    - [`input_split`](#input_split)
    - [`input_get`](#input_get)
    - [`input_csi_dispatch_winops`](#input_csi_dispatch_winops)
    - [`input_reply`](#input_reply)
    - [`input_report_current_theme`](#input_report_current_theme)
    - [`input_restore_state`](#input_restore_state)
    - [`input_csi_dispatch_rm`](#input_csi_dispatch_rm)
    - [`input_csi_dispatch_rm_private`](#input_csi_dispatch_rm_private)
    - [`input_save_state`](#input_save_state)
    - [`input_csi_dispatch_sgr`](#input_csi_dispatch_sgr)
    - [`input_csi_dispatch_sm`](#input_csi_dispatch_sm)
    - [`input_csi_dispatch_sm_private`](#input_csi_dispatch_sm_private)
    - [`input_csi_dispatch_sm_graphics`](#input_csi_dispatch_sm_graphics)


---
### input_csi_dispatch_rm<!-- {{#callable:input_csi_dispatch_rm}} -->
The `input_csi_dispatch_rm` function processes Control Sequence Introducer (CSI) 'Reset Mode' commands.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input processing, including the current state and parameters.
- **Control Flow**:
    - Iterates over the parameters in the `param_list` of the `input_ctx` structure.
    - For each parameter, it retrieves its value using the [`input_get`](#input_get) function.
    - Based on the retrieved value, it performs specific actions such as clearing or setting modes in the screen context.
    - If an unknown parameter is encountered, it logs a debug message.
- **Output**:
    - The function does not return a value but modifies the screen state based on the parameters processed.
- **Functions called**:
    - [`input_get`](#input_get)


---
### input_csi_dispatch_rm_private<!-- {{#callable:input_csi_dispatch_rm_private}} -->
The `input_csi_dispatch_rm_private` function processes private mode reset (RM) control sequences for terminal input.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input processing, including the current state and parameters.
- **Control Flow**:
    - Iterates over the `param_list_len` of the `input_ctx` to process each parameter.
    - For each parameter, it retrieves the value using [`input_get`](#input_get) and checks against known control sequence cases.
    - Depending on the parameter value, it calls various `screen_write_mode_clear` or `screen_write_cursormove` functions to modify the terminal's state.
    - If an unknown parameter is encountered, it logs a debug message.
- **Output**:
    - The function does not return a value but modifies the terminal's state based on the control sequences processed.
- **Functions called**:
    - [`input_get`](#input_get)


---
### input_csi_dispatch_sm<!-- {{#callable:input_csi_dispatch_sm}} -->
The `input_csi_dispatch_sm` function processes Control Sequence Introducer (CSI) Set Mode commands.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
- **Control Flow**:
    - Iterates over the parameters in the `param_list` of the `input_ctx` structure.
    - For each parameter, it retrieves its value using the [`input_get`](#input_get) function.
    - Based on the retrieved value, it executes specific actions such as setting or clearing modes.
    - If an unknown parameter is encountered, it logs a debug message.
- **Output**:
    - The function does not return a value but modifies the state of the screen by setting or clearing modes based on the parameters processed.
- **Functions called**:
    - [`input_get`](#input_get)


---
### input_csi_dispatch_sm_private<!-- {{#callable:input_csi_dispatch_sm_private}} -->
The `input_csi_dispatch_sm_private` function processes private mode setting commands from the input context.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the current input context, including parameters and state.
- **Control Flow**:
    - Iterates over the `param_list_len` of the `input_ctx` to process each parameter.
    - For each parameter, it retrieves the value using the [`input_get`](#input_get) function.
    - Based on the retrieved value, it executes specific screen writing functions to set modes or clear modes.
    - If an unknown parameter is encountered, it logs a debug message.
- **Output**:
    - The function does not return a value but modifies the screen state based on the parameters processed.
- **Functions called**:
    - [`input_get`](#input_get)


---
### input_csi_dispatch_sm_graphics<!-- {{#callable:input_csi_dispatch_sm_graphics}} -->
Handles the CSI (Control Sequence Introducer) 'SM' (Set Mode) graphics commands.
- **Inputs**:
    - `ictx`: Pointer to the `input_ctx` structure that contains the context for input processing.
- **Control Flow**:
    - Checks if the `param_list_len` in `ictx` is greater than 3, and returns early if true.
    - Retrieves the first three parameters using [`input_get`](#input_get) function.
    - If the first parameter `n` is 1 and the second parameter `m` is either 1, 2, or 4, it sends a reply with the number of Sixel colour registers.
    - Otherwise, it sends a different reply based on the value of `n` and the third parameter `o`.
- **Output**:
    - Sends a response to the terminal based on the parameters received, specifically related to Sixel graphics support.
- **Functions called**:
    - [`input_get`](#input_get)
    - [`input_reply`](#input_reply)


---
### input_csi_dispatch_winops<!-- {{#callable:input_csi_dispatch_winops}} -->
The `input_csi_dispatch_winops` function processes window operations commands received in a Control Sequence Introducer (CSI) format.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input processing, including the current window pane and screen write context.
- **Control Flow**:
    - The function retrieves the current window pane from the input context.
    - It enters a loop to read input parameters using the [`input_get`](#input_get) function until no more parameters are available.
    - For each parameter, it uses a switch statement to handle specific window operation commands based on the parameter value.
    - Certain commands may require additional parameters, which are fetched in sequence, and if any required parameter is missing, the function returns early.
    - The function sends responses back to the terminal using the [`input_reply`](#input_reply) function for specific commands that require feedback.
- **Output**:
    - The function does not return a value but sends responses to the terminal based on the window operation commands processed.
- **Functions called**:
    - [`input_get`](#input_get)
    - [`input_reply`](#input_reply)


---
### input_csi_dispatch_sgr_256_do<!-- {{#callable:input_csi_dispatch_sgr_256_do}} -->
The `input_csi_dispatch_sgr_256_do` function processes 256-color SGR (Select Graphic Rendition) commands to set foreground, background, or underline colors.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
    - `fgbg`: An integer indicating whether the command is for foreground (38) or background (48) color.
    - `c`: An integer representing the color index, which should be between 0 and 255, or -1 for default.
- **Control Flow**:
    - The function first checks if the color index `c` is -1 or greater than 255.
    - If `c` is -1 and `fgbg` is 38, it sets the foreground color to a default value (8).
    - If `c` is -1 and `fgbg` is 48, it sets the background color to a default value (8).
    - If `c` is valid (0 to 255), it sets the foreground or background color based on the value of `fgbg` by applying a color flag (COLOUR_FLAG_256).
    - If `fgbg` is 58, it sets the underline color.
- **Output**:
    - The function returns 1 to indicate successful processing of the SGR command.


---
### input_csi_dispatch_sgr_256<!-- {{#callable:input_csi_dispatch_sgr_256}} -->
Handles the dispatch of CSI SGR (Select Graphic Rendition) sequences for 256 colors.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
    - `fgbg`: An integer indicating whether the operation is for foreground (38) or background (48) color.
    - `i`: A pointer to an unsigned integer that represents the index of the current parameter being processed.
- **Control Flow**:
    - Calls [`input_get`](#input_get) to retrieve the next color value from the input context based on the current index.
    - If the retrieved color value is valid, it calls [`input_csi_dispatch_sgr_256_do`](#input_csi_dispatch_sgr_256_do) to apply the color to the appropriate attribute (foreground or background).
    - If the operation is successful, it increments the index pointer to process the next parameter.
- **Output**:
    - The function does not return a value but modifies the color attributes of the `input_ctx` based on the SGR parameters.
- **Functions called**:
    - [`input_get`](#input_get)
    - [`input_csi_dispatch_sgr_256_do`](#input_csi_dispatch_sgr_256_do)


---
### input_csi_dispatch_sgr_rgb_do<!-- {{#callable:input_csi_dispatch_sgr_rgb_do}} -->
The `input_csi_dispatch_sgr_rgb_do` function processes RGB color values for foreground, background, or underline attributes in a terminal context.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
    - `fgbg`: An integer indicating whether to set the foreground (38), background (48), or underline (58) color.
    - `r`: An integer representing the red component of the RGB color, expected to be in the range 0-255.
    - `g`: An integer representing the green component of the RGB color, expected to be in the range 0-255.
    - `b`: An integer representing the blue component of the RGB color, expected to be in the range 0-255.
- **Control Flow**:
    - The function first checks if the red, green, and blue values are within the valid range (0-255). If any value is out of range, it returns 0.
    - If the `fgbg` parameter is 38, it sets the foreground color of the grid cell using the `colour_join_rgb` function with the provided RGB values.
    - If the `fgbg` parameter is 48, it sets the background color of the grid cell using the same function.
    - If the `fgbg` parameter is 58, it sets the underline color of the grid cell.
    - Finally, the function returns 1 to indicate successful processing.
- **Output**:
    - The function returns an integer: 0 if the RGB values are invalid, and 1 if the color setting was successful.


---
### input_csi_dispatch_sgr_rgb<!-- {{#callable:input_csi_dispatch_sgr_rgb}} -->
The `input_csi_dispatch_sgr_rgb` function processes RGB color settings for terminal graphics.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
    - `fgbg`: An integer indicating whether the RGB values are for foreground (38) or background (48) color.
    - `i`: A pointer to an unsigned integer that serves as an index for reading RGB values from the parameter list.
- **Control Flow**:
    - The function retrieves the red, green, and blue color values from the input context using the [`input_get`](#input_get) function.
    - It checks if the RGB values are valid (between 0 and 255) using the [`input_csi_dispatch_sgr_rgb_do`](#input_csi_dispatch_sgr_rgb_do) helper function.
    - If the RGB values are valid, it updates the foreground or background color in the `grid_cell` structure of the input context.
- **Output**:
    - The function does not return a value but modifies the color attributes of the `grid_cell` in the `input_ctx` structure based on the RGB values provided.
- **Functions called**:
    - [`input_get`](#input_get)
    - [`input_csi_dispatch_sgr_rgb_do`](#input_csi_dispatch_sgr_rgb_do)


---
### input_csi_dispatch_sgr_colon<!-- {{#callable:input_csi_dispatch_sgr_colon}} -->
The `input_csi_dispatch_sgr_colon` function processes SGR (Select Graphic Rendition) parameters separated by colons.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
    - `i`: An unsigned integer representing the index of the parameter in the `param_list` to be processed.
- **Control Flow**:
    - Initializes an array `p` to hold parsed parameters, setting all values to -1.
    - Duplicates the string parameter from `param_list[i]` and splits it by colons into individual components.
    - Converts each non-empty component into an integer and stores it in the `p` array, checking for errors.
    - If the first parameter is 4, it processes the second parameter to set various underscore attributes on the grid cell.
    - If the first parameter is not 4, it checks for color parameters (38, 48, 58) and processes them accordingly.
    - Calls helper functions to handle RGB or 256 color settings based on the parsed parameters.
- **Output**:
    - The function modifies the attributes of the `grid_cell` in the `input_ctx` based on the parsed SGR parameters, affecting text rendering in the terminal.
- **Functions called**:
    - [`input_csi_dispatch_sgr_rgb_do`](#input_csi_dispatch_sgr_rgb_do)
    - [`input_csi_dispatch_sgr_256_do`](#input_csi_dispatch_sgr_256_do)


---
### input_csi_dispatch_sgr<!-- {{#callable:input_csi_dispatch_sgr}} -->
The `input_csi_dispatch_sgr` function processes SGR (Select Graphic Rendition) control sequences to modify text attributes and colors in a terminal.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the current input context, including the parameter list and the grid cell to be modified.
- **Control Flow**:
    - If the `param_list_len` of `ictx` is zero, the function copies the default grid cell attributes to the current cell and returns.
    - The function iterates over the parameters in `ictx->param_list`.
    - For each parameter, if it is a string type, it calls [`input_csi_dispatch_sgr_colon`](#input_csi_dispatch_sgr_colon) to handle it.
    - If the parameter is a number, it retrieves its value using [`input_get`](#input_get).
    - If the value indicates a color change (38, 48, or 58), it processes RGB or 256 color values accordingly.
    - For other values, it modifies the attributes of the grid cell based on the SGR codes, such as setting foreground/background colors or text attributes like bold, dim, or underline.
- **Output**:
    - The function modifies the attributes of the grid cell referenced by `ictx` based on the SGR parameters provided, affecting how text is rendered in the terminal.
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
    - `ictx`: A pointer to the `input_ctx` structure that holds the state and context for input processing.
- **Control Flow**:
    - Logs the function entry using `log_debug`.
    - Calls [`input_clear`](#input_clear) to reset the input context state.
    - Calls [`input_start_timer`](#input_start_timer) to start a timer for the input context.
    - Clears the `INPUT_LAST` flag in the `flags` member of the input context.
- **Output**:
    - The function does not return a value; it modifies the state of the `input_ctx` structure to prepare for processing a DCS sequence.
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
    - If the `INPUT_DISCARD` flag is set in `ictx`, it logs a debug message and returns 0.
    - If the first byte of the input buffer is 'q' and there are no intermediate characters, it attempts to parse a sixel image if `ENABLE_SIXEL` is defined.
    - Retrieves the `allow-passthrough` option from the window's options; if it is not set, the function returns 0.
    - Logs the input buffer content for debugging.
    - If the input buffer length is greater than or equal to the length of the prefix 'tmux;', it checks if the input starts with this prefix.
    - If it does, it writes the raw string from the input buffer (excluding the prefix) to the screen context, considering passthrough options.
- **Output**:
    - The function returns 0, indicating successful processing of the input, with potential side effects on the screen output depending on the input content.
- **Functions called**:
    - [`input_split`](#input_split)
    - [`input_get`](#input_get)


---
### input_enter_osc<!-- {{#callable:input_enter_osc}} -->
The `input_enter_osc` function initializes the input context for processing OSC (Operating System Command) sequences.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the current input context.
- **Control Flow**:
    - Logs the function entry using `log_debug`.
    - Calls [`input_clear`](#input_clear) to reset the input context.
    - Calls [`input_start_timer`](#input_start_timer) to start a timer for input processing.
    - Clears the `INPUT_LAST` flag in the `flags` member of the input context.
- **Output**:
    - The function does not return a value; it modifies the state of the input context to prepare for OSC input processing.
- **Functions called**:
    - [`input_clear`](#input_clear)
    - [`input_start_timer`](#input_start_timer)


---
### input_exit_osc<!-- {{#callable:input_exit_osc}} -->
The `input_exit_osc` function processes the end of an Operating System Command (OSC) sequence, handling various options based on the input.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for the input being processed, including the input buffer and associated window pane.
- **Control Flow**:
    - Checks if the `INPUT_DISCARD` flag is set in the `ictx` structure; if so, the function returns immediately.
    - Validates that the input length is greater than 0 and that the first character of the input buffer is a digit (0-9); if not, it returns.
    - Logs the current function name and the input buffer content for debugging purposes.
    - Parses the numeric option from the input buffer until a semicolon or null terminator is encountered.
    - Switches on the parsed option value to determine the appropriate action to take, including setting the window title, handling colors, or invoking specific OSC handlers.
    - For options that require further processing, calls the corresponding helper functions (e.g., [`input_osc_4`](#input_osc_4), [`input_osc_8`](#input_osc_8), etc.).
    - Logs an unknown option if the parsed option does not match any expected values.
- **Output**:
    - The function does not return a value but performs actions based on the parsed OSC option, such as updating the window title or modifying screen attributes.
- **Functions called**:
    - [`input_osc_4`](#input_osc_4)
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
The `input_enter_apc` function initializes the input context for Application Programming Commands (APC) mode.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
- **Control Flow**:
    - Logs the function entry using `log_debug`.
    - Calls [`input_clear`](#input_clear) to reset the input context state.
    - Starts a timer for input processing using [`input_start_timer`](#input_start_timer).
    - Clears the `INPUT_LAST` flag in the `flags` member of the `input_ctx` structure.
- **Output**:
    - The function does not return a value; it modifies the state of the `input_ctx` structure to prepare for APC input processing.
- **Functions called**:
    - [`input_clear`](#input_clear)
    - [`input_start_timer`](#input_start_timer)


---
### input_exit_apc<!-- {{#callable:input_exit_apc}} -->
The `input_exit_apc` function processes the end of an Application Protocol Command (APC) sequence by setting the terminal title.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input processing, including the input buffer and associated window pane.
- **Control Flow**:
    - Checks if the `INPUT_DISCARD` flag is set in the `ictx` structure; if so, the function returns immediately without processing.
    - Logs the current function name and the content of the input buffer.
    - Calls `screen_set_title` to set the terminal title using the content of the input buffer.
    - If the title is successfully set and the associated window pane (`wp`) is not NULL, it sends a notification to update the pane title and redraws the window borders and status.
- **Output**:
    - The function does not return a value; it modifies the terminal's title and may trigger UI updates in the associated window pane.


---
### input_enter_rename<!-- {{#callable:input_enter_rename}} -->
The `input_enter_rename` function initializes the input context for renaming operations.
- **Inputs**:
    - None
- **Control Flow**:
    - Logs the function entry using `log_debug`.
    - Calls [`input_clear`](#input_clear) to reset the input context.
    - Starts a timer for input operations using [`input_start_timer`](#input_start_timer).
    - Clears the `INPUT_LAST` flag from the context's flags.
- **Output**:
    - The function does not return a value; it modifies the state of the input context for renaming.
- **Functions called**:
    - [`input_clear`](#input_clear)
    - [`input_start_timer`](#input_start_timer)


---
### input_exit_rename<!-- {{#callable:input_exit_rename}} -->
The `input_exit_rename` function handles the termination of a rename operation for a window pane.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for the input operation, including the window pane and input buffer.
- **Control Flow**:
    - Checks if the window pane (`wp`) is NULL; if so, it returns immediately.
    - Checks if the `INPUT_DISCARD` flag is set in the context; if so, it returns immediately.
    - Checks if renaming is allowed by retrieving the `allow-rename` option; if not allowed, it returns.
    - Logs the current input buffer for debugging purposes.
    - Validates the UTF-8 encoding of the input buffer; if invalid, it returns.
    - Retrieves the associated window from the window pane.
    - If the input length is zero, it checks for the `automatic-rename` option and clears the window name if necessary.
    - If the input length is greater than zero, it sets the window name to the input buffer and disables automatic renaming.
    - Calls functions to redraw the window borders and update the status of the window.
- **Output**:
    - The function does not return a value but modifies the window name based on the input buffer and updates the window's visual representation.


---
### input_top_bit_set<!-- {{#callable:input_top_bit_set}} -->
The `input_top_bit_set` function processes the top bit of input characters for UTF-8 encoding.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
- **Control Flow**:
    - Clears the `INPUT_LAST` flag from the `flags` field of the `input_ctx` structure.
    - Checks if UTF-8 processing has started; if not, attempts to open a new UTF-8 sequence with the current character.
    - If the UTF-8 sequence is already started, it appends the character to the existing sequence.
    - Handles the result of the UTF-8 appending: if more data is needed, it returns; if an error occurs, it resets the UTF-8 state.
    - If a complete UTF-8 character is formed, it logs the character details, copies the character data to the cell, and adds it to the screen write context.
    - Updates the last character processed and sets the `INPUT_LAST` flag.
- **Output**:
    - The function returns 0 upon successful processing of the input character, indicating no errors.


---
### input_osc_colour_reply<!-- {{#callable:input_osc_colour_reply}} -->
The `input_osc_colour_reply` function sends a color reply to the terminal based on the provided color value.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
    - `n`: An unsigned integer representing the color index or identifier.
    - `c`: An integer representing the color value, which may be transformed into RGB format.
- **Control Flow**:
    - If `c` is not -1, it is converted to RGB format using `colour_force_rgb`.
    - If the resulting `c` is -1, the function returns early without sending a reply.
    - The RGB components are extracted from `c` using `colour_split_rgb`.
    - The termination character for the reply is determined based on the `input_end` state of `ictx`.
    - The formatted color reply is sent to the terminal using [`input_reply`](#input_reply).
- **Output**:
    - The function does not return a value; instead, it sends a formatted string to the terminal that specifies the RGB color.
- **Functions called**:
    - [`input_reply`](#input_reply)


---
### input_osc_4<!-- {{#callable:input_osc_4}} -->
The `input_osc_4` function processes OSC 4 escape sequences to set or query color palette entries.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
    - `p`: A string containing the parameters for the OSC 4 command, which may include color indices and values.
- **Control Flow**:
    - The function duplicates the input string `p` into `copy` and initializes `s` to point to it.
    - It enters a loop that continues until `s` is NULL or points to an empty string.
    - Within the loop, it parses an index from `s` using `strtol` and checks for a semicolon delimiter.
    - If the index is valid (0 to 255), it checks if the next part of the string is a query ('?') or a color value.
    - If it's a query, it retrieves the color from the palette and sends a reply using [`input_osc_colour_reply`](#input_osc_colour_reply).
    - If it's a color value, it attempts to parse it using `colour_parseX11` and sets it in the palette if valid.
    - If any errors occur during parsing, a flag `bad` is set, and the loop breaks.
    - After processing, if `bad` is set, a debug message is logged, and if any colors were changed, a full redraw of the screen is requested.
- **Output**:
    - The function does not return a value but may modify the color palette and trigger a screen redraw if colors are changed.
- **Functions called**:
    - [`input_osc_colour_reply`](#input_osc_colour_reply)


---
### input_osc_8<!-- {{#callable:input_osc_8}} -->
The `input_osc_8` function processes OSC 8 escape sequences to handle hyperlinks.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input processing.
    - `p`: A string containing the parameters of the OSC 8 sequence, which may include an optional id and a URI.
- **Control Flow**:
    - The function initializes local variables for hyperlink context and cell attributes.
    - It iterates through the input string `p` to extract parameters, specifically looking for an 'id' and a URI.
    - If a valid URI is found, it stores the hyperlink in the context's hyperlink structure.
    - If the URI is empty, it sets the link attribute of the grid cell to 0.
    - If any errors occur during parsing, it jumps to the 'bad' label to handle the error.
- **Output**:
    - The function does not return a value but modifies the hyperlink state in the input context and logs debug information about the hyperlink.


---
### input_osc_10<!-- {{#callable:input_osc_10}} -->
The `input_osc_10` function handles the OSC 10 escape sequence for setting and querying the foreground color.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input handling.
    - `p`: A string representing the parameters passed with the OSC 10 sequence, which can be a color value or a query indicator ('?').
- **Control Flow**:
    - The function first checks if the input string `p` is equal to '?', indicating a request for the current foreground color.
    - If `p` is '?', it retrieves the foreground color from the associated `window_pane` or defaults if not set, and sends a reply with the current color.
    - If `p` is not '?', it attempts to parse the color value from the string.
    - If the color parsing fails, it logs an error and exits.
    - If successful, it updates the foreground color in the `palette` of the `input_ctx` and marks the `window_pane` as changed, triggering a full redraw of the screen.
- **Output**:
    - The function does not return a value but sends a reply to the terminal with the current foreground color if queried, or updates the foreground color in the context if a valid color is provided.
- **Functions called**:
    - [`input_osc_colour_reply`](#input_osc_colour_reply)


---
### input_osc_110<!-- {{#callable:input_osc_110}} -->
The `input_osc_110` function resets the foreground color of the terminal to a default value.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
    - `p`: A string parameter that is expected to be empty for the function to proceed.
- **Control Flow**:
    - The function first checks if the string pointed to by `p` is not empty; if it is not, the function returns immediately.
    - If the `palette` member of the `input_ctx` structure is not NULL, it sets the foreground color to a default value (8).
    - If the `window_pane` associated with the `input_ctx` is not NULL, it updates the pane's flags to indicate that the style has changed.
    - Finally, it triggers a full redraw of the screen using the `screen_write_fullredraw` function.
- **Output**:
    - The function does not return a value; it modifies the state of the terminal's foreground color and triggers a redraw if applicable.


---
### input_osc_11<!-- {{#callable:input_osc_11}} -->
The `input_osc_11` function handles the OSC 11 escape sequence for setting or querying the background color.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input handling.
    - `p`: A string representing the parameters passed with the OSC 11 sequence, which can be a color value or a query indicator ('?').
- **Control Flow**:
    - The function first checks if the input string `p` is equal to '?'. If it is, it retrieves the current background color from the associated window pane and sends a reply with that color.
    - If `p` is not '?', it attempts to parse the color value from the string using `colour_parseX11`. If parsing fails, it logs an error and exits.
    - If the color is successfully parsed and the palette is not NULL, it updates the background color in the palette and marks the window pane as changed, triggering a full redraw of the screen.
- **Output**:
    - The function does not return a value but modifies the state of the input context and potentially sends a reply to the terminal with the current background color.
- **Functions called**:
    - [`input_osc_colour_reply`](#input_osc_colour_reply)


---
### input_osc_111<!-- {{#callable:input_osc_111}} -->
The `input_osc_111` function resets the background color of the terminal to a default value.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input handling.
    - `p`: A string parameter that is expected to be empty for this function to execute its logic.
- **Control Flow**:
    - The function first checks if the input string `p` is not empty; if it is not, the function returns immediately without making any changes.
    - If the `palette` member of the `input_ctx` structure is not NULL, it sets the background color of the palette to 8.
    - If the `window_pane` associated with the `input_ctx` is not NULL, it updates the pane's flags to indicate that the style and theme have changed.
    - Finally, it triggers a full redraw of the screen using the `screen_write_fullredraw` function.
- **Output**:
    - The function does not return a value; it modifies the state of the terminal's color palette and triggers a redraw of the screen.


---
### input_osc_12<!-- {{#callable:input_osc_12}} -->
The `input_osc_12` function handles the OSC 12 escape sequence for setting or querying the cursor color.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input handling.
    - `p`: A string representing the parameters passed with the OSC 12 sequence, which can be a color value or a query indicator ('?').
- **Control Flow**:
    - The function first checks if the input string `p` is equal to '?'.
    - If it is, and if the `window_pane` associated with `ictx` is not NULL, it retrieves the current cursor color.
    - If the cursor color is -1, it falls back to the default cursor color.
    - It then calls [`input_osc_colour_reply`](#input_osc_colour_reply) to send the current cursor color back to the terminal.
    - If `p` is not '?', it attempts to parse the color value from `p` using `colour_parseX11`.
    - If the parsing fails (returns -1), it logs a debug message and exits.
    - If successful, it sets the cursor color using `screen_set_cursor_colour`.
- **Output**:
    - The function does not return a value but modifies the cursor color in the terminal or replies with the current cursor color if queried.
- **Functions called**:
    - [`input_osc_colour_reply`](#input_osc_colour_reply)


---
### input_osc_112<!-- {{#callable:input_osc_112}} -->
The `input_osc_112` function resets the cursor color to the default when no arguments are provided.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
    - `p`: A string representing the parameters for the OSC (Operating System Command) sequence.
- **Control Flow**:
    - The function checks if the input string `p` is empty.
    - If `p` is empty, it calls `screen_set_cursor_colour` with the context's screen and -1 to reset the cursor color.
- **Output**:
    - The function does not return a value; it modifies the cursor color on the screen based on the input parameters.


---
### input_osc_133<!-- {{#callable:input_osc_133}} -->
The `input_osc_133` function processes OSC 133 escape sequences to set flags on a specific grid line.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
    - `p`: A pointer to a string containing the parameters for the OSC 133 command.
- **Control Flow**:
    - The function retrieves the grid associated with the current input context.
    - It calculates the current line number based on the cursor's vertical position.
    - If the calculated line number exceeds the grid's height, the function returns early.
    - The function retrieves the grid line structure for the calculated line number.
    - It checks the first character of the input string to determine the action: if it's 'A', it sets the `GRID_LINE_START_PROMPT` flag; if it's 'C', it sets the `GRID_LINE_START_OUTPUT` flag.
- **Output**:
    - The function does not return a value; it modifies the flags of the specified grid line based on the input parameters.


---
### input_osc_52<!-- {{#callable:input_osc_52}} -->
The `input_osc_52` function handles the OSC 52 escape sequence for clipboard operations.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that contains the context for input handling.
    - `p`: A pointer to a string containing the parameters for the OSC 52 command.
- **Control Flow**:
    - Checks if the `window_pane` associated with the input context is NULL; if so, it returns immediately.
    - Retrieves the clipboard setting state from global options; if it is not set to 2, the function returns.
    - Looks for a semicolon in the input string to separate the flags from the clipboard data; if not found, it returns.
    - Logs the debug information about the input string after the semicolon.
    - Iterates through the characters in the input string before the semicolon to collect unique flags that are allowed.
    - If the input string after the semicolon is a question mark, it retrieves the current clipboard content and sends it back.
    - If the input string contains data, it decodes the base64 encoded data and sets it to the clipboard.
    - Starts a screen write context, sets the selection with the decoded data, and notifies the pane of the clipboard update.
- **Output**:
    - The function does not return a value but modifies the clipboard content and sends a response back to the terminal.
- **Functions called**:
    - [`input_reply_clipboard`](#input_reply_clipboard)


---
### input_osc_104<!-- {{#callable:input_osc_104}} -->
The `input_osc_104` function processes OSC 104 escape sequences to unset specified color palette entries.
- **Inputs**:
    - `ictx`: A pointer to the `input_ctx` structure that holds the context for input processing.
    - `p`: A string containing the parameters for the OSC 104 command, which specifies the indices of the color palette entries to unset.
- **Control Flow**:
    - If the input string `p` is empty, the function clears the color palette and triggers a full redraw of the screen.
    - The function duplicates the input string `p` for processing.
    - It enters a loop to parse the input string, extracting indices for the color palette entries.
    - For each index, it checks if it is valid (between 0 and 255) and attempts to unset the corresponding color in the palette.
    - If any index is invalid, a debug message is logged.
    - If any palette entry was successfully unset, a flag is set to indicate that a redraw is needed.
    - Finally, if any errors were encountered, a debug message is logged, and if redraw is needed, the screen is redrawn.
- **Output**:
    - The function does not return a value but modifies the color palette and may trigger a screen redraw based on the input parameters.


---
### input_reply_clipboard<!-- {{#callable:input_reply_clipboard}} -->
The `input_reply_clipboard` function encodes a given buffer into Base64 and sends it to a terminal as a clipboard update.
- **Inputs**:
    - `bev`: A pointer to a `bufferevent` structure representing the event buffer to which the clipboard data will be written.
    - `buf`: A pointer to a character array containing the data to be encoded and sent to the clipboard.
    - `len`: The length of the data in `buf` that needs to be encoded.
    - `end`: A pointer to a character array that indicates the end of the clipboard update command.
- **Control Flow**:
    - The function first checks if `buf` is not NULL and `len` is not zero.
    - If the length of the buffer exceeds a certain threshold, the function returns early to prevent overflow.
    - The function calculates the required output length for the Base64 encoded data and allocates memory for it.
    - It then encodes the input buffer `buf` into Base64 format using `b64_ntop`.
    - If the encoding fails, it frees the allocated memory and returns.
    - The function writes the initial clipboard command sequence to the `bufferevent`.
    - If the encoded output length is not zero, it writes the encoded data to the `bufferevent`.
    - Finally, it writes the end sequence to the `bufferevent` and frees the allocated memory.
- **Output**:
    - The function does not return a value; instead, it sends the Base64 encoded clipboard data to the specified `bufferevent`.


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
    - The function retrieves the current theme of the window pane using `window_pane_get_theme`.
    - It then checks the theme value and responds accordingly: if the theme is `THEME_DARK`, it sends a response indicating this theme; if it is `THEME_LIGHT`, it sends a different response; if the theme is `THEME_UNKNOWN`, it does nothing.
- **Output**:
    - The function does not return a value but sends a response back to the input context's event, indicating the current theme of the window pane.
- **Functions called**:
    - [`input_reply`](#input_reply)


# Function Declarations (Public API)

---
### input_split<!-- {{#callable_declaration:input_split}} -->
Parses and splits input parameters from a buffer.
- **Description**: This function processes the input parameters stored in the `param_buf` of the given `input_ctx` structure, splitting them into individual parameters based on the ';' delimiter. It categorizes each parameter as a string, number, or missing, and stores them in the `param_list`. This function should be called when the input buffer is populated and needs to be parsed into discrete parameters. It returns an error if a parameter cannot be converted to a number or if the parameter list exceeds its capacity.
- **Inputs**:
    - `ictx`: A pointer to an `input_ctx` structure containing the input buffer and parameter list. Must not be null. The function modifies the `param_list` and `param_list_len` fields of this structure.
- **Output**: Returns 0 on success, or -1 if an error occurs (e.g., invalid number conversion or parameter list overflow).
- **See also**: [`input_split`](#input_split)  (Implementation)


---
### input_get<!-- {{#callable_declaration:input_get}} -->
Retrieve a numeric parameter from the input context.
- **Description**: This function retrieves a numeric parameter from the input context's parameter list based on the specified index. It is used to obtain parameters that are expected to be numbers, with provisions for default values and minimum constraints. If the parameter at the given index is missing or is a string, the function returns a default value. If the retrieved numeric value is less than the specified minimum, the function returns the minimum value instead. This function is useful in scenarios where parameters are optional or have default values, and it ensures that the returned value is always within a valid range.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure that contains the parameter list. Must not be null.
    - `validx`: An unsigned integer representing the index of the parameter to retrieve. Must be less than the length of the parameter list; otherwise, the default value is returned.
    - `minval`: An integer specifying the minimum allowable value for the parameter. If the retrieved value is less than this, minval is returned.
    - `defval`: An integer specifying the default value to return if the parameter is missing or is a string.
- **Output**: Returns an integer which is the parameter value, the default value, or the minimum value, depending on the conditions described.
- **See also**: [`input_get`](#input_get)  (Implementation)


---
### input_set_state<!-- {{#callable_declaration:input_set_state}} -->
Transitions the input context to a new state.
- **Description**: This function is used to change the current state of an input context to a new state as specified by the input transition. It should be called when a state transition is required in the input parsing process. The function will handle any necessary exit actions for the current state and entry actions for the new state. It is important to ensure that the input context and transition provided are valid and properly initialized before calling this function.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the current input context. Must not be null and should be properly initialized.
    - `itr`: A pointer to an input_transition structure that specifies the new state to transition to. Must not be null and should be properly initialized.
- **Output**: None
- **See also**: [`input_set_state`](#input_set_state)  (Implementation)


---
### input_reset_cell<!-- {{#callable_declaration:input_reset_cell}} -->
Resets the input cell state to default values.
- **Description**: Use this function to reset the state of the input cell within the input context to its default values. This is typically necessary when reinitializing or clearing the input state to ensure that the input cell does not retain any previous state or settings. It is important to call this function when you need to ensure that the input cell is in a known, default state before processing new input.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the input context. Must not be null, as the function operates directly on the provided context to reset its cell state.
- **Output**: None
- **See also**: [`input_reset_cell`](#input_reset_cell)  (Implementation)


---
### input_report_current_theme<!-- {{#callable_declaration:input_report_current_theme}} -->
Reports the current theme of the input context's window pane.
- **Description**: This function is used to report the current theme of a window pane associated with the given input context. It sends a specific response based on the theme: a dark theme results in a "\033[?997;1n" response, a light theme results in a "\033[?997;2n" response, and an unknown theme results in no response. This function should be called when there is a need to query and report the theme status of a window pane, typically in response to a specific terminal query.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the input context. This parameter must not be null, and it is expected to have a valid window pane (wp) associated with it. The function does not handle null or invalid input contexts.
- **Output**: None
- **See also**: [`input_report_current_theme`](#input_report_current_theme)  (Implementation)


---
### input_osc_4<!-- {{#callable_declaration:input_osc_4}} -->
Handles OSC 4 sequences to set or query terminal color palette entries.
- **Description**: This function processes OSC 4 sequences, which are used to set or query color palette entries in a terminal. It should be called with a valid input context and a string representing the OSC 4 sequence. The function parses the sequence to either set a color at a specified index in the palette or query the current color. If a query is made, a response is sent back with the current color value. The function handles invalid sequences by logging an error and may trigger a full screen redraw if the palette is modified. It is important to ensure that the input context and the string are correctly initialized and valid before calling this function.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the current input context. Must not be null and should be properly initialized with a valid color palette.
    - `p`: A constant character pointer representing the OSC 4 sequence string. It should be a null-terminated string and must not be null. The string should be formatted correctly as an OSC 4 sequence, otherwise, the function will log an error.
- **Output**: None
- **See also**: [`input_osc_4`](#input_osc_4)  (Implementation)


---
### input_osc_8<!-- {{#callable_declaration:input_osc_8}} -->
Handles OSC 8 sequences for embedding hyperlinks.
- **Description**: This function processes OSC 8 sequences to embed hyperlinks in terminal output. It extracts parameters and the URI from the input string, and associates the URI with a grid cell in the input context. If an 'id' parameter is present, it is used to uniquely identify the hyperlink; otherwise, the hyperlink is anonymous. The function must be called with a valid input context and a non-null string representing the OSC 8 sequence. If the input is malformed, the function logs an error and does not modify the grid cell.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the current input context. Must not be null. The function uses this context to access the grid cell and hyperlink storage.
    - `p`: A pointer to a null-terminated string containing the OSC 8 sequence. Must not be null. The string should follow the format 'id=<id>;uri', where 'id' is optional and 'uri' is the hyperlink.
- **Output**: None
- **See also**: [`input_osc_8`](#input_osc_8)  (Implementation)


---
### input_osc_10<!-- {{#callable_declaration:input_osc_10}} -->
Handles OSC 10 sequence for setting or querying the foreground color.
- **Description**: This function processes the OSC 10 sequence, which is used to set or query the foreground color of a terminal. When the input string `p` is "?", it queries the current foreground color and sends a reply with the color information. If `p` contains a valid color specification, it sets the foreground color to the specified value. This function should be called when handling OSC 10 sequences in terminal emulation. It requires a valid `input_ctx` structure and a non-null string `p`. If the color specification is invalid, the function logs a debug message and takes no further action.
- **Inputs**:
    - `ictx`: A pointer to an `input_ctx` structure representing the current input context. Must not be null.
    - `p`: A constant character pointer representing the OSC 10 sequence parameter. It can be "?" to query the current color or a valid color specification. Must not be null.
- **Output**: None
- **See also**: [`input_osc_10`](#input_osc_10)  (Implementation)


---
### input_osc_11<!-- {{#callable_declaration:input_osc_11}} -->
Handles OSC 11 sequences for setting or querying the background color.
- **Description**: This function processes OSC 11 sequences to either set or query the background color of a terminal pane. When the input string `p` is "?", it queries the current background color and sends a reply with the color information. If `p` contains a valid color specification, it sets the background color to the specified value. The function expects a valid `input_ctx` structure and a non-null string `p`. If the color specification is invalid, the function logs a debug message and takes no further action. This function should be used within a terminal emulator context where OSC sequences are handled.
- **Inputs**:
    - `ictx`: A pointer to an `input_ctx` structure representing the input context. It must be properly initialized and must not be null.
    - `p`: A constant character pointer representing the OSC 11 parameter string. It must not be null. If the string is "?", the function queries the current background color. Otherwise, it should be a valid color specification.
- **Output**: None
- **See also**: [`input_osc_11`](#input_osc_11)  (Implementation)


---
### input_osc_12<!-- {{#callable_declaration:input_osc_12}} -->
Sets or queries the cursor color based on the input string.
- **Description**: This function is used to set or query the cursor color in a terminal context. It should be called with a valid input context and a string representing the desired color or a query. If the string is '?', the function will query the current cursor color and send a reply. If the string is a valid color, it will set the cursor color accordingly. Invalid color strings are ignored, and no changes are made. This function is typically used in response to OSC 12 sequences in terminal emulation.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the current input context. Must not be null.
    - `p`: A string representing the color to set or '?' to query the current cursor color. If the string is not a valid color or '?', the function will not perform any action.
- **Output**: None
- **See also**: [`input_osc_12`](#input_osc_12)  (Implementation)


---
### input_osc_52<!-- {{#callable_declaration:input_osc_52}} -->
Handles OSC 52 sequences for clipboard operations.
- **Description**: This function processes OSC 52 sequences to interact with the clipboard in a terminal environment. It is used to either query the current clipboard content or set new clipboard data, depending on the input parameters. The function should be called when an OSC 52 sequence is detected, and it requires a valid input context and a string parameter. The function checks for the presence of a window pane and the appropriate clipboard setting before proceeding. It handles both querying and setting clipboard data, with the latter involving base64 decoding of the input string.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the current input context. Must not be null and should be properly initialized with a valid window pane.
    - `p`: A pointer to a null-terminated string containing the OSC 52 sequence parameters. The string must include a semicolon-separated list of flags followed by the base64-encoded clipboard data or a '?' to query the clipboard.
- **Output**: None
- **See also**: [`input_osc_52`](#input_osc_52)  (Implementation)


---
### input_osc_104<!-- {{#callable_declaration:input_osc_104}} -->
Clears or resets specific color palette entries.
- **Description**: This function is used to clear or reset specific entries in a color palette associated with a terminal input context. It is typically called when handling OSC 104 sequences in terminal emulation. If the input string is empty, the entire palette is cleared. Otherwise, it parses the input string for indices, resetting each specified index to its default state. The function must be called with a valid input context and a non-null string. If any invalid indices or characters are encountered, they are logged, and the function continues processing the rest of the string. A full screen redraw is triggered if any palette entries are changed.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the terminal input context. Must not be null.
    - `p`: A string containing indices of the color palette to reset, separated by semicolons. Can be empty to reset the entire palette. Must not be null.
- **Output**: None
- **See also**: [`input_osc_104`](#input_osc_104)  (Implementation)


---
### input_osc_110<!-- {{#callable_declaration:input_osc_110}} -->
Resets the foreground color in the input context's palette.
- **Description**: This function is used to reset the foreground color in the input context's palette to its default value. It should be called when the input string is empty, indicating a reset command. If the input context has a valid palette, the foreground color is set to the default index, and a full redraw of the screen is triggered. This function is typically used in terminal emulation to handle OSC 110 sequences for resetting colors.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the input context. It must not be null and should contain a valid palette if color resetting is desired.
    - `p`: A pointer to a constant character string. It should be an empty string to trigger the reset of the foreground color. Non-empty strings are ignored.
- **Output**: None
- **See also**: [`input_osc_110`](#input_osc_110)  (Implementation)


---
### input_osc_111<!-- {{#callable_declaration:input_osc_111}} -->
Resets the background color of the input context to default.
- **Description**: This function is used to reset the background color of the input context's palette to its default value. It should be called when a reset of the background color is required, such as when handling specific OSC (Operating System Command) sequences. The function checks if the input string is empty before proceeding. If the input context has a valid palette, it sets the background color to the default and triggers a full redraw of the screen context. This function does not return any value and does not handle invalid input strings.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the input context. It must not be null and should have a valid palette if color changes are to be applied.
    - `p`: A pointer to a constant character string. It is expected to be an empty string for the function to proceed with resetting the background color. Non-empty strings are ignored.
- **Output**: None
- **See also**: [`input_osc_111`](#input_osc_111)  (Implementation)


---
### input_osc_112<!-- {{#callable_declaration:input_osc_112}} -->
Resets the cursor color to its default value.
- **Description**: This function is used to reset the cursor color to its default setting. It should be called when there is a need to revert any changes made to the cursor color, ensuring that no arguments are passed to it. This function is typically used in scenarios where the cursor color has been modified and needs to be restored to its original state.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure, which must be valid and properly initialized. The caller retains ownership and must ensure it is not null.
    - `p`: A pointer to a constant character string. It must point to an empty string (i.e., the first character must be the null terminator '\0'). If this condition is not met, the function will not perform any action.
- **Output**: None
- **See also**: [`input_osc_112`](#input_osc_112)  (Implementation)


---
### input_osc_133<!-- {{#callable_declaration:input_osc_133}} -->
Sets flags on the current grid line based on the input character.
- **Description**: This function is used to modify the flags of the current line in a grid based on a given character input. It should be called with a valid input context and a character pointer. The function checks if the current line is within the bounds of the grid and then sets specific flags on the line depending on the character provided. It is important to ensure that the input context and character pointer are valid and that the character is either 'A' or 'C' to have an effect.
- **Inputs**:
    - `ictx`: A pointer to an input context structure. Must not be null and should be properly initialized before calling this function.
    - `p`: A pointer to a character string. The first character is used to determine which flag to set. Must not be null and should point to a valid character ('A' or 'C').
- **Output**: None
- **See also**: [`input_osc_133`](#input_osc_133)  (Implementation)


---
### input_clear<!-- {{#callable_declaration:input_clear}} -->
Clears the input context buffers and resets state flags.
- **Description**: This function is used to reset the input context by clearing various buffers and resetting state flags. It should be called when the input context needs to be reinitialized or reset to a known state. This function stops any active timers associated with the input context and clears intermediate, parameter, and input buffers. It also resets the input end state and clears specific flags. This function does not handle any invalid input as it assumes the input context is valid.
- **Inputs**:
    - `ictx`: A pointer to a valid input_ctx structure. Must not be null, and the caller retains ownership. The function assumes the structure is properly initialized.
- **Output**: None
- **See also**: [`input_clear`](#input_clear)  (Implementation)


---
### input_ground<!-- {{#callable_declaration:input_ground}} -->
Resets the input context to the ground state.
- **Description**: This function is used to reset the input context to its initial ground state, which is a neutral state in the input parsing process. It should be called when the input context needs to be cleared of any accumulated data or when transitioning to a new input sequence. The function ensures that any pending events are canceled and the input buffer is resized to its initial size if it has grown beyond that. This function does not return any value and does not handle invalid input context pointers.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the input context. This parameter must not be null, and the caller retains ownership of the memory. The function assumes the input context is valid and initialized.
- **Output**: None
- **See also**: [`input_ground`](#input_ground)  (Implementation)


---
### input_enter_dcs<!-- {{#callable_declaration:input_enter_dcs}} -->
Prepares the input context for DCS (Device Control String) processing.
- **Description**: This function is used to prepare the input context for processing a Device Control String (DCS) by clearing any existing input state, starting a timer to handle timeouts, and resetting specific flags. It should be called when entering a DCS state to ensure the input context is correctly initialized for handling DCS sequences. This function does not return any value and does not modify any input parameters.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the input context. This parameter must not be null, and the caller retains ownership. The function will modify the state of this structure to prepare it for DCS processing.
- **Output**: None
- **See also**: [`input_enter_dcs`](#input_enter_dcs)  (Implementation)


---
### input_enter_osc<!-- {{#callable_declaration:input_enter_osc}} -->
Enter the OSC (Operating System Command) state for input processing.
- **Description**: This function is used to transition the input context into the OSC (Operating System Command) state. It should be called when the input parser needs to handle OSC sequences. The function clears any existing input state, starts a timer to handle timeouts, and resets specific flags. It is typically used in terminal emulation contexts where OSC sequences are part of the input stream.
- **Inputs**:
    - `ictx`: A pointer to an `input_ctx` structure representing the current input context. This parameter must not be null, and the caller retains ownership. The function modifies the state of this context to prepare for OSC input processing.
- **Output**: None
- **See also**: [`input_enter_osc`](#input_enter_osc)  (Implementation)


---
### input_exit_osc<!-- {{#callable_declaration:input_exit_osc}} -->
Handles the termination of an OSC sequence in the input context.
- **Description**: This function processes the termination of an Operating System Command (OSC) sequence within the input context. It should be called when an OSC sequence is completed, either by a string terminator (ST) or a bell character (BEL). The function checks if the input should be discarded based on the context flags and validates the input buffer for a valid OSC option number. Depending on the option number, it performs various actions such as setting the window title, handling color settings, or other OSC-specific operations. This function assumes that the input context has been properly initialized and that the input buffer contains a valid OSC sequence.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the current input context. Must not be null. The function expects this context to be properly initialized and contain a valid OSC sequence in its input buffer. If the INPUT_DISCARD flag is set, the function will return immediately without processing.
- **Output**: None
- **See also**: [`input_exit_osc`](#input_exit_osc)  (Implementation)


---
### input_enter_apc<!-- {{#callable_declaration:input_enter_apc}} -->
Enter the APC state in the input context.
- **Description**: This function is used to transition the input context into the APC (Application Program Command) state. It should be called when the input parser needs to handle an APC sequence. The function clears any existing input state, starts a timer to handle timeouts, and resets specific flags in the input context. It is typically used in terminal emulation scenarios where handling of APC sequences is required.
- **Inputs**:
    - `ictx`: A pointer to an `input_ctx` structure representing the input context. It must not be null, and the caller retains ownership. The function modifies the state and flags of this context.
- **Output**: None
- **See also**: [`input_enter_apc`](#input_enter_apc)  (Implementation)


---
### input_exit_apc<!-- {{#callable_declaration:input_exit_apc}} -->
Handles the termination of an APC input sequence.
- **Description**: This function is used to process the end of an Application Program Command (APC) input sequence. It should be called when an APC sequence is completed. If the input context has the INPUT_DISCARD flag set, the function will return immediately without performing any actions. Otherwise, it attempts to set the screen title using the input buffer. If successful and a window pane is associated with the context, it triggers notifications and redraws the window borders and status. This function should be used in environments where APC sequences are handled, and it assumes that the input context and associated structures are properly initialized.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the input context. It must not be null and should be properly initialized. The function checks the INPUT_DISCARD flag in this context and uses the input_buf to set the screen title.
- **Output**: None
- **See also**: [`input_exit_apc`](#input_exit_apc)  (Implementation)


---
### input_enter_rename<!-- {{#callable_declaration:input_enter_rename}} -->
Prepares the input context for renaming operations.
- **Description**: This function is used to prepare the input context for handling rename operations. It should be called when entering a state where renaming is expected. The function clears any existing input state, starts a timer to manage input duration, and resets specific flags in the input context. It is important to ensure that the input context is properly initialized before calling this function to avoid undefined behavior.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the input context. Must not be null. The caller retains ownership and is responsible for ensuring the context is valid and initialized.
- **Output**: None
- **See also**: [`input_enter_rename`](#input_enter_rename)  (Implementation)


---
### input_exit_rename<!-- {{#callable_declaration:input_exit_rename}} -->
Handles the renaming of a window based on input context.
- **Description**: This function is used to rename a window associated with a given input context, provided certain conditions are met. It should be called when a rename operation is needed, typically after processing input that may affect the window's name. The function checks if the window pane is valid, if renaming is allowed, and if the input buffer contains valid UTF-8 data. If the input buffer is empty, it may reset the window's name to an empty string if automatic renaming is disabled. Otherwise, it sets the window's name to the content of the input buffer and disables automatic renaming. It also triggers a redraw of the window borders and updates the window status.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the input context. It must not be null and should have a valid window pane (wp) associated with it. The function checks various flags and options within this context to determine if renaming is allowed and what actions to take.
- **Output**: None
- **See also**: [`input_exit_rename`](#input_exit_rename)  (Implementation)


---
### input_print<!-- {{#callable_declaration:input_print}} -->
Outputs a character to the screen based on the input context.
- **Description**: This function is used to output a character to the screen using the provided input context. It manages character attributes, including character set and UTF-8 data, and updates the screen write context accordingly. This function should be called when a character needs to be printed to the screen, ensuring that the input context is properly initialized and contains valid data. It handles character attributes and ensures that the last character is stored for potential repetition.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure that contains the current input context, including character data and attributes. Must not be null and should be properly initialized before calling this function.
- **Output**: Returns 0 to indicate successful execution.
- **See also**: [`input_print`](#input_print)  (Implementation)


---
### input_intermediate<!-- {{#callable_declaration:input_intermediate}} -->
Appends a character to the intermediate buffer or sets a discard flag.
- **Description**: This function is used to append a character to the intermediate buffer within an input context. It should be called when a character needs to be added to the buffer during input parsing. If the buffer is full, the function sets a flag to indicate that further input should be discarded. This function must be used in contexts where the input context is properly initialized and managed.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the input context. Must not be null. The function assumes that the input context is properly initialized and managed by the caller.
- **Output**: Returns 0 to indicate successful execution.
- **See also**: [`input_intermediate`](#input_intermediate)  (Implementation)


---
### input_parameter<!-- {{#callable_declaration:input_parameter}} -->
Appends a character to the parameter buffer or sets a discard flag.
- **Description**: This function is used to append a character to the parameter buffer within an input context. It should be called when a new character needs to be added to the parameter buffer. If the buffer is full, the function sets a flag to indicate that further input should be discarded. This function does not handle any input validation or error checking beyond checking the buffer's capacity.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure, which must not be null. The caller retains ownership and is responsible for ensuring the structure is properly initialized before calling this function.
- **Output**: Returns 0 to indicate successful execution, but the return value is not used to convey any specific information.
- **See also**: [`input_parameter`](#input_parameter)  (Implementation)


---
### input_input<!-- {{#callable_declaration:input_input}} -->
Appends a character to the input buffer of the input context.
- **Description**: This function is used to append a character to the input buffer within an input context structure. It ensures that there is enough space in the buffer to accommodate the new character by expanding the buffer size if necessary. If the buffer size exceeds a predefined maximum, the function sets a discard flag and stops appending. This function should be called when a new character needs to be added to the input buffer, and it assumes that the input context has been properly initialized.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the input context. It must not be null, and the input context should be properly initialized before calling this function.
- **Output**: None
- **See also**: [`input_input`](#input_input)  (Implementation)


---
### input_c0_dispatch<!-- {{#callable_declaration:input_c0_dispatch}} -->
Dispatches C0 control characters for terminal input processing.
- **Description**: This function processes C0 control characters from the input context, handling various terminal control sequences such as backspace, line feed, and carriage return. It is typically used within a terminal emulator or similar environment to interpret and act on control characters received from input. The function expects a valid input context and modifies the screen state accordingly. It should be called as part of a larger input handling routine that processes terminal input.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the current input context. This must be a valid, non-null pointer, and the structure should be properly initialized before calling this function. The function uses this context to determine the current character to process and to update the screen state.
- **Output**: Returns 0 to indicate successful processing of the input character. The function modifies the screen state and input context based on the character processed.
- **See also**: [`input_c0_dispatch`](#input_c0_dispatch)  (Implementation)


---
### input_esc_dispatch<!-- {{#callable_declaration:input_esc_dispatch}} -->
Dispatches an escape sequence based on the current input context.
- **Description**: This function processes an escape sequence from the input context and performs the corresponding action based on the escape command type. It should be called when an escape sequence is detected in the input stream. The function checks if the input context is marked for discarding and returns immediately if so. It uses a lookup table to find the appropriate action for the escape sequence and executes it, affecting the screen state or input context as necessary. This function does not handle invalid escape sequences beyond logging them.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the current input context. Must not be null. The function expects this context to be properly initialized and may modify its state.
- **Output**: Always returns 0, indicating successful processing of the escape sequence, even if the sequence is unknown or discarded.
- **See also**: [`input_esc_dispatch`](#input_esc_dispatch)  (Implementation)


---
### input_csi_dispatch<!-- {{#callable_declaration:input_csi_dispatch}} -->
Dispatches a CSI (Control Sequence Introducer) command based on the input context.
- **Description**: This function processes a CSI command from the input context, executing the appropriate action based on the command type. It should be called when a CSI sequence is detected in the input stream. The function expects the input context to be properly initialized and populated with the current input data. It handles various CSI commands, such as cursor movements, screen modifications, and mode settings. The function does not perform any action if the INPUT_DISCARD flag is set in the input context.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the current input context. It must be non-null and properly initialized. The function uses this context to determine the CSI command and its parameters. If the INPUT_DISCARD flag is set, the function will return immediately without processing the command.
- **Output**: Returns 0 after processing the CSI command, with potential side effects on the screen or input context state.
- **See also**: [`input_csi_dispatch`](#input_csi_dispatch)  (Implementation)


---
### input_csi_dispatch_rm<!-- {{#callable_declaration:input_csi_dispatch_rm}} -->
Handles the CSI RM (Reset Mode) control sequence.
- **Description**: This function processes the CSI RM (Reset Mode) control sequence by iterating over the list of parameters in the input context and applying the appropriate screen mode changes. It is typically used in terminal emulation to reset specific modes based on the parameters provided. The function expects a valid input context with a populated parameter list and modifies the screen write context accordingly. It handles known parameters by clearing or setting specific screen modes and logs unknown parameters for debugging purposes.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the current input context. It must not be null and should contain a valid parameter list. The function will iterate over this list to determine which modes to reset.
- **Output**: None
- **See also**: [`input_csi_dispatch_rm`](#input_csi_dispatch_rm)  (Implementation)


---
### input_csi_dispatch_rm_private<!-- {{#callable_declaration:input_csi_dispatch_rm_private}} -->
Handles private CSI RM (Reset Mode) sequences for terminal input.
- **Description**: This function processes private CSI RM sequences, which are used to reset various terminal modes. It iterates over a list of parameters and applies the corresponding mode reset actions to the terminal screen context. This function is typically called as part of a larger input handling mechanism in a terminal emulator. It expects a valid input context with a populated parameter list and modifies the screen state based on the parameters provided.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the current input context. It must be non-null and contain a valid parameter list. The function will read from this context to determine which modes to reset.
- **Output**: None
- **See also**: [`input_csi_dispatch_rm_private`](#input_csi_dispatch_rm_private)  (Implementation)


---
### input_csi_dispatch_sm<!-- {{#callable_declaration:input_csi_dispatch_sm}} -->
Sets specific screen modes based on CSI parameters.
- **Description**: This function processes a list of parameters from a CSI (Control Sequence Introducer) sequence and sets or clears specific screen modes accordingly. It is typically used in terminal emulation to handle mode settings like insert mode or cursor visibility. The function should be called with a valid input context containing the parameters to be processed. It does not return a value but modifies the screen context based on the parameters.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure that contains the context for input processing, including the list of parameters to be processed. Must not be null.
- **Output**: None
- **See also**: [`input_csi_dispatch_sm`](#input_csi_dispatch_sm)  (Implementation)


---
### input_csi_dispatch_sm_private<!-- {{#callable_declaration:input_csi_dispatch_sm_private}} -->
Sets various terminal modes based on private CSI parameters.
- **Description**: This function processes a list of private CSI (Control Sequence Introducer) parameters and sets the corresponding terminal modes. It is typically used in terminal emulation to handle specific control sequences that modify terminal behavior, such as cursor visibility, mouse mode, and screen mode. The function should be called with a valid input context that contains the parameters to be processed. It assumes that the input context has been properly initialized and populated with parameters. The function does not return a value but modifies the terminal state based on the parameters provided.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure that contains the context for input processing, including the list of parameters to be processed. Must not be null. The function expects ictx->param_list_len to indicate the number of parameters and ictx->param_list to contain the parameters.
- **Output**: None
- **See also**: [`input_csi_dispatch_sm_private`](#input_csi_dispatch_sm_private)  (Implementation)


---
### input_csi_dispatch_sm_graphics<!-- {{#callable_declaration:input_csi_dispatch_sm_graphics}} -->
Handles CSI graphics mode settings for input context.
- **Description**: This function processes the CSI (Control Sequence Introducer) graphics mode settings for an input context, specifically when the SIXEL graphics extension is enabled. It should be used when handling terminal input sequences that involve graphics mode settings. The function checks the number of parameters and responds with appropriate terminal control sequences based on the parameter values. It is important to ensure that the input context is properly initialized and that the SIXEL extension is enabled before calling this function.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the input context. This parameter must not be null and should be properly initialized. The function expects the input context to have a list of parameters that it will process. If the parameter list length exceeds three, the function will return without performing any action.
- **Output**: None
- **See also**: [`input_csi_dispatch_sm_graphics`](#input_csi_dispatch_sm_graphics)  (Implementation)


---
### input_csi_dispatch_winops<!-- {{#callable_declaration:input_csi_dispatch_winops}} -->
Handles window operations based on CSI sequences.
- **Description**: This function processes Control Sequence Introducer (CSI) sequences related to window operations within a terminal emulator context. It should be called when a CSI sequence that pertains to window operations is detected. The function interprets various CSI codes to perform actions such as querying window size, setting window titles, and other window-related operations. It requires a valid input context that includes the current window pane and screen context. The function does not return a value but may send replies to the terminal or modify the screen state based on the CSI commands processed.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the current input context. This must not be null and should contain valid references to the associated window pane and screen context. The function will handle invalid or unsupported CSI sequences gracefully, typically by ignoring them.
- **Output**: None
- **See also**: [`input_csi_dispatch_winops`](#input_csi_dispatch_winops)  (Implementation)


---
### input_csi_dispatch_sgr_256<!-- {{#callable_declaration:input_csi_dispatch_sgr_256}} -->
Handles CSI SGR 256 color sequences.
- **Description**: This function processes CSI SGR sequences specifically for 256 color support, updating the color attributes of the input context based on the provided parameters. It should be used when parsing terminal input sequences that specify 256 color settings. The function expects valid input parameters and will adjust the index pointer if a valid color is processed.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the current input context. Must not be null.
    - `fgbg`: An integer indicating whether the color is for the foreground (38) or background (48). Must be either 38 or 48.
    - `i`: A pointer to an unsigned integer representing the current index in the parameter list. The function may increment this index if a valid color is processed. Must not be null.
- **Output**: None
- **See also**: [`input_csi_dispatch_sgr_256`](#input_csi_dispatch_sgr_256)  (Implementation)


---
### input_csi_dispatch_sgr_rgb<!-- {{#callable_declaration:input_csi_dispatch_sgr_rgb}} -->
Handles RGB color setting for SGR sequences in a terminal input context.
- **Description**: This function processes RGB color settings for Select Graphic Rendition (SGR) sequences in a terminal input context. It is used to set the foreground, background, or underline color based on the provided RGB values. The function should be called when an SGR sequence with RGB color parameters is encountered. It updates the color settings in the input context and advances the parameter index if the RGB values are valid.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the current input context. This must not be null and should be properly initialized before calling the function.
    - `fgbg`: An integer indicating whether the color is for the foreground (38), background (48), or underline (58). It must be one of these values.
    - `i`: A pointer to an unsigned integer representing the current index in the parameter list. The function will increment this index if the RGB values are valid.
- **Output**: None
- **See also**: [`input_csi_dispatch_sgr_rgb`](#input_csi_dispatch_sgr_rgb)  (Implementation)


---
### input_csi_dispatch_sgr<!-- {{#callable_declaration:input_csi_dispatch_sgr}} -->
Parses and applies Select Graphic Rendition (SGR) parameters to a grid cell.
- **Description**: This function processes a list of Select Graphic Rendition (SGR) parameters from the input context and applies the corresponding text attributes and colors to the current grid cell. It should be called when SGR parameters are received as part of a control sequence in a terminal emulator. The function handles various SGR codes, including those for text attributes like bold, italic, and underline, as well as color settings for foreground, background, and underline colors. It supports both standard and extended color codes, including 256-color and RGB color specifications. The function expects the input context to be properly initialized and populated with SGR parameters before invocation.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure that contains the current input context, including the list of SGR parameters to be processed. Must not be null. The function assumes that the input context is correctly initialized and contains valid SGR parameters.
- **Output**: None
- **See also**: [`input_csi_dispatch_sgr`](#input_csi_dispatch_sgr)  (Implementation)


---
### input_dcs_dispatch<!-- {{#callable_declaration:input_dcs_dispatch}} -->
Handles Device Control String (DCS) sequences in terminal input.
- **Description**: This function processes Device Control String (DCS) sequences received as part of terminal input. It is typically called within a terminal emulator or similar environment to handle specific DCS sequences, such as those prefixed with 'tmux;'. The function checks for certain conditions, such as whether the input should be discarded or if passthrough is allowed, and processes the input accordingly. It is important to ensure that the input context (`ictx`) is properly initialized and that the associated window pane (`wp`) is not null before calling this function.
- **Inputs**:
    - `ictx`: A pointer to an `input_ctx` structure representing the current input context. This must be properly initialized and must not be null. The function expects `ictx->wp` to be non-null for processing.
- **Output**: Returns an integer status code, typically 0, indicating the result of the processing. The function may also modify the screen context or write raw strings based on the input.
- **See also**: [`input_dcs_dispatch`](#input_dcs_dispatch)  (Implementation)


---
### input_top_bit_set<!-- {{#callable_declaration:input_top_bit_set}} -->
Processes a UTF-8 character in the input context.
- **Description**: This function is used to handle a UTF-8 character within an input context, typically as part of a terminal emulator or similar application. It should be called when a character with the top bit set is encountered, indicating a potential UTF-8 sequence. The function manages the state of UTF-8 processing, appending bytes to a UTF-8 sequence until it is complete or an error occurs. It must be called with a valid input context that has been properly initialized. The function does not handle invalid input gracefully, so the caller must ensure that the input context is in a valid state before calling.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the current input context. This must be a valid, non-null pointer, and the input context should be properly initialized before calling this function.
- **Output**: Always returns 0, indicating that the function does not produce a meaningful return value. The input context may be modified to reflect the state of UTF-8 processing.
- **See also**: [`input_top_bit_set`](#input_top_bit_set)  (Implementation)


---
### input_end_bel<!-- {{#callable_declaration:input_end_bel}} -->
Sets the input end state to BEL.
- **Description**: This function is used to set the input end state of the input context to BEL, which is typically used to signify the end of an OSC (Operating System Command) sequence terminated by a BEL character. It should be called when processing input sequences that are expected to end with a BEL. This function does not perform any validation on the input context and assumes it is valid.
- **Inputs**:
    - `ictx`: A pointer to an input_ctx structure representing the input context. Must not be null. The caller retains ownership and is responsible for ensuring the context is valid.
- **Output**: Returns 0 to indicate successful completion.
- **See also**: [`input_end_bel`](#input_end_bel)  (Implementation)


