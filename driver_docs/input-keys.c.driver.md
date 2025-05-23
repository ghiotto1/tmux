# Purpose
This C source code file is part of the tmux terminal multiplexer and is responsible for translating key codes into sequences suitable for sending to applications running within tmux panes. The file defines a structure and functions to manage a tree of key mappings, which are used to convert key codes into escape sequences that can be understood by terminal applications. The code includes a list of default key mappings for various keys, such as function keys, arrow keys, and keypad keys, and provides functionality to handle key modifiers like Shift, Meta, and Ctrl.

The file is not an executable but rather a component of the tmux codebase, likely intended to be compiled and linked with other parts of the tmux application. It defines internal functions and data structures, such as `input_key_entry` and `input_key_tree`, to manage key mappings and perform key translation. The code also includes functions to handle mouse events and output them in a format compatible with terminal applications. The file does not define public APIs or external interfaces, as it is focused on internal key handling logic within the tmux application.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### input_key_tree
- **Type**: `struct input_key_tree`
- **Description**: The `input_key_tree` is a global variable of type `struct input_key_tree`, which is a red-black tree data structure used to store and manage input key entries. Each entry in this tree is represented by the `input_key_entry` structure, which contains a key code and associated data. The tree is initialized using the `RB_INITIALIZER` macro, which sets up the tree for subsequent operations.
- **Use**: This variable is used to store and manage a collection of input key entries, allowing for efficient lookup and insertion of key codes and their corresponding data.


---
### input_key_defaults
- **Type**: ``struct input_key_entry[]``
- **Description**: The `input_key_defaults` is a static array of `struct input_key_entry` that defines default key mappings for various keyboard keys and their corresponding escape sequences. This array includes mappings for paste keys, function keys, arrow keys, keypad keys, and keys with embedded modifiers. Each entry in the array consists of a key code and its associated data, which is typically an escape sequence string.
- **Use**: This variable is used to initialize a tree of input keys that translate key codes into sequences suitable for applications running in a terminal pane.


---
### input_key_modifiers
- **Type**: ``key_code[]``
- **Description**: The `input_key_modifiers` is a static constant array of type `key_code` that defines a set of key modifier combinations. Each element in the array represents a combination of key modifiers such as SHIFT, META, and CTRL, using bitwise OR operations to combine them. This array is used to map key codes to their corresponding modifier states.
- **Use**: This variable is used to determine the modifier state of key codes when building the input key tree for translating key codes into output sequences.


# Data Structures

---
### input_key_entry
- **Type**: `struct`
- **Members**:
    - `key`: Represents the key code associated with the input key entry.
    - `data`: Points to a string that contains the data or sequence associated with the key code.
    - `entry`: A Red-Black tree entry used to link this structure into a Red-Black tree.
- **Description**: The `input_key_entry` structure is used to represent an entry in a Red-Black tree that maps key codes to their corresponding data sequences. This structure is part of a system that translates key codes into sequences suitable for sending to an application running in a terminal pane. Each entry in the tree consists of a key code, a pointer to the associated data string, and a Red-Black tree entry for maintaining the tree structure. The Red-Black tree is used to efficiently store and retrieve key entries based on their key codes.


# Functions

---
### input_key_cmp<!-- {{#callable:input_key_cmp}} -->
The `input_key_cmp` function compares two `input_key_entry` structures based on their `key` values and returns an integer indicating their relative order.
- **Inputs**:
    - `ike1`: A pointer to the first `input_key_entry` structure to be compared.
    - `ike2`: A pointer to the second `input_key_entry` structure to be compared.
- **Control Flow**:
    - Check if the `key` of `ike1` is less than the `key` of `ike2`; if true, return -1.
    - Check if the `key` of `ike1` is greater than the `key` of `ike2`; if true, return 1.
    - If neither condition is met, return 0, indicating the keys are equal.
- **Output**:
    - An integer: -1 if `ike1`'s key is less than `ike2`'s key, 1 if greater, and 0 if they are equal.


---
### input_key_get<!-- {{#callable:input_key_get}} -->
The `input_key_get` function searches for a key in a red-black tree and returns the corresponding entry if found.
- **Inputs**:
    - `key`: A `key_code` representing the key to be searched in the red-black tree.
- **Control Flow**:
    - A local `input_key_entry` structure is initialized with the provided key.
    - The function calls `RB_FIND` to search for the entry in the `input_key_tree` using the initialized structure.
    - The result of `RB_FIND`, which is either a pointer to the found entry or NULL if not found, is returned.
- **Output**:
    - A pointer to the `input_key_entry` structure if the key is found in the tree, otherwise NULL.


---
### input_key_split2<!-- {{#callable:input_key_split2}} -->
The `input_key_split2` function converts a Unicode character into its UTF-8 byte representation and stores it in a destination buffer.
- **Inputs**:
    - `c`: An unsigned integer representing the Unicode character to be converted.
    - `dst`: A pointer to an unsigned character array where the UTF-8 byte representation of the character will be stored.
- **Control Flow**:
    - Check if the input character `c` is greater than 0x7f (i.e., if it requires more than one byte in UTF-8).
    - If `c` is greater than 0x7f, calculate the first byte by shifting `c` right by 6 bits and OR-ing with 0xc0, and store it in `dst[0]`.
    - Calculate the second byte by AND-ing `c` with 0x3f and OR-ing with 0x80, and store it in `dst[1]`.
    - Return 2, indicating that two bytes were used.
    - If `c` is not greater than 0x7f, store `c` directly in `dst[0]`.
    - Return 1, indicating that one byte was used.
- **Output**:
    - The function returns the number of bytes used to represent the character in UTF-8, which is either 1 or 2.


---
### input_key_build<!-- {{#callable:input_key_build}} -->
The `input_key_build` function constructs a tree of input key entries by processing default key entries and their modifiers, and logs the constructed keys.
- **Inputs**:
    - None
- **Control Flow**:
    - Iterate over each entry in the `input_key_defaults` array.
    - For each entry, check if the key has modifiers using a bitwise AND operation with `KEYC_BUILD_MODIFIERS`.
    - If the key does not have modifiers, insert the entry into the `input_key_tree` and continue to the next entry.
    - If the key has modifiers, iterate over the `input_key_modifiers` array starting from index 2.
    - For each modifier, create a new key by combining the base key with the modifier, and duplicate the data string, replacing the underscore with the modifier index.
    - Allocate memory for a new `input_key_entry`, set its key and data, and insert it into the `input_key_tree`.
    - After processing all entries, iterate over the `input_key_tree` and log each key and its associated data.
- **Output**:
    - The function does not return a value; it modifies the global `input_key_tree` by inserting key entries.


---
### input_key_pane<!-- {{#callable:input_key_pane}} -->
The `input_key_pane` function processes a key event for a specific window pane, handling mouse events separately and logging the key if logging is enabled.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure representing the pane where the key event is to be processed.
    - `key`: A `key_code` representing the key event to be processed.
    - `m`: A pointer to a `mouse_event` structure, which may be NULL, representing a mouse event associated with the key event.
- **Control Flow**:
    - Check if logging is enabled by calling `log_get_level()` and log the key event using `log_debug` if it is.
    - Determine if the key event is a mouse event using `KEYC_IS_MOUSE(key)`.
    - If the key is a mouse event and the mouse event `m` is not NULL, check if the mouse event is associated with the current pane by comparing `m->wp` with `wp->id`.
    - If the mouse event is associated with the current pane, call `input_key_mouse(wp, m)` to handle the mouse event and return 0.
    - If the key is not a mouse event, call `input_key(wp->screen, wp->event, key)` to process the key event and return its result.
- **Output**:
    - Returns 0 if the key is a mouse event and is handled, otherwise returns the result of [`input_key`](#input_key) for non-mouse key events.
- **Functions called**:
    - [`input_key_mouse`](#input_key_mouse)
    - [`input_key`](#input_key)


---
### input_key_write<!-- {{#callable:input_key_write}} -->
The `input_key_write` function logs a debug message and writes a specified data buffer to a bufferevent.
- **Inputs**:
    - `from`: A string representing the source or context from which the function is called, used for logging purposes.
    - `bev`: A pointer to a `bufferevent` structure where the data will be written.
    - `data`: A pointer to the data buffer that needs to be written to the bufferevent.
    - `size`: The size of the data buffer to be written, specified as a `size_t` type.
- **Control Flow**:
    - The function begins by logging a debug message that includes the source context and the data to be written, formatted as a string with a specified size.
    - It then calls `bufferevent_write` to write the data buffer to the specified bufferevent.
- **Output**:
    - The function does not return any value; it performs logging and writes data to a bufferevent as side effects.


---
### input_key_extended<!-- {{#callable:input_key_extended}} -->
The `input_key_extended` function encodes and writes an extended key escape sequence to a buffer event, based on the key's modifiers and a configured output format.
- **Inputs**:
    - `bev`: A pointer to a `bufferevent` structure where the encoded key sequence will be written.
    - `key`: A `key_code` representing the key to be encoded, which may include modifier flags.
- **Control Flow**:
    - The function begins by determining the modifier character based on the key's modifier flags using a switch statement.
    - If the key is a Unicode key, it converts the key to a wide character using UTF-8 conversion functions; if conversion fails, it returns -1.
    - If the key is not Unicode, it masks the key to remove modifier flags.
    - The function checks the global option 'extended-keys-format' to decide between two output formats for the escape sequence.
    - It formats the escape sequence into a temporary buffer using `xsnprintf`, either as '\033[27;modifier;key~' or '\033[key;modifieru'.
    - The formatted escape sequence is written to the buffer event using [`input_key_write`](#input_key_write).
    - The function returns 0 to indicate success.
- **Output**:
    - The function returns 0 on success, or -1 if it encounters an error such as an unsupported key modifier combination or a failed Unicode conversion.
- **Functions called**:
    - [`input_key_write`](#input_key_write)


---
### input_key_vt10x<!-- {{#callable:input_key_vt10x}} -->
The `input_key_vt10x` function translates a key code into a corresponding output sequence for a terminal, handling various key modifiers and control codes.
- **Inputs**:
    - `bev`: A pointer to a `bufferevent` structure used for writing the output key sequence.
    - `key`: A `key_code` representing the key input to be translated and output.
- **Control Flow**:
    - Log the input key code for debugging purposes.
    - Check if the key has the META modifier and write the escape character '\033' if true.
    - If the key is a Unicode key, convert it to UTF-8 and write it to the buffer, then return 0.
    - Extract the base key by masking with `KEYC_MASK_KEY` and handle special cases for TAB, CR, and LF by removing the CTRL modifier.
    - If the key has the CTRL modifier, attempt to map it to a C0 control code using a predefined map or calculate it directly; return -1 if mapping fails.
    - Log the output key code for debugging purposes.
    - Write the final key code to the buffer, ensuring it is within the 7-bit ASCII range, and return 0.
- **Output**:
    - The function returns 0 on successful translation and writing of the key sequence, or -1 if an error occurs during control code mapping.
- **Functions called**:
    - [`input_key_write`](#input_key_write)


---
### input_key_mode1<!-- {{#callable:input_key_mode1}} -->
The `input_key_mode1` function processes a key code to determine if it should be translated into a VT10x key sequence based on specific modifier conditions.
- **Inputs**:
    - `bev`: A pointer to a `bufferevent` structure used for buffered I/O operations.
    - `key`: A `key_code` representing the key input to be processed.
- **Control Flow**:
    - Log the function entry and the key value for debugging purposes.
    - Check if the key has only the Meta modifier and, if so, call [`input_key_vt10x`](#input_key_vt10x) to process it as a VT10x key.
    - Extract the base key value by masking out modifiers using `KEYC_MASK_KEY`.
    - Check if the key has the Ctrl modifier and if the base key is one of the specified characters or within certain ranges.
    - If the above condition is met, call [`input_key_vt10x`](#input_key_vt10x) to process the key as a VT10x key.
    - If none of the conditions are met, return -1 indicating the key is not handled.
- **Output**:
    - Returns an integer, either the result of [`input_key_vt10x`](#input_key_vt10x) if the key is processed, or -1 if the key does not meet the conditions for processing.
- **Functions called**:
    - [`input_key_vt10x`](#input_key_vt10x)


---
### input_key<!-- {{#callable:input_key}} -->
The `input_key` function processes a key code and translates it into a suitable output sequence for an application running in a terminal pane.
- **Inputs**:
    - `s`: A pointer to a `screen` structure representing the terminal screen context.
    - `bev`: A pointer to a `bufferevent` structure used for buffered I/O operations.
    - `key`: A `key_code` representing the key input to be processed.
- **Control Flow**:
    - Check if the key is a mouse event and return 0 if true, as mouse keys need a pane.
    - If the key is a literal key, write it directly to the buffer and return 0.
    - Check if the key is a backspace, remap it if necessary, and write the appropriate sequence to the buffer.
    - Check if the key is a backtab and adjust the key code based on the screen mode.
    - Handle trivial cases where the key is a 7-bit key or a UTF-8 key with no modifiers, writing the appropriate sequence to the buffer.
    - Look up the key in the standard VT10x key tree, adjusting for keypad and cursor modes, and write the corresponding sequence if found.
    - Ignore internal function key codes and return 0 if the key falls within certain ranges.
    - If no built-in key sequence is found, construct an extended key sequence based on the client mode and write it to the buffer.
- **Output**:
    - The function returns 0 after processing the key, indicating successful handling, or calls other functions to handle extended key sequences.
- **Functions called**:
    - [`input_key_write`](#input_key_write)
    - [`input_key_get`](#input_key_get)
    - [`input_key_extended`](#input_key_extended)
    - [`input_key_mode1`](#input_key_mode1)
    - [`input_key_vt10x`](#input_key_vt10x)


---
### input_key_get_mouse<!-- {{#callable:input_key_get_mouse}} -->
The `input_key_get_mouse` function processes mouse events and generates an appropriate escape sequence for the terminal based on the screen mode and mouse event details.
- **Inputs**:
    - `s`: A pointer to a `screen` structure representing the current screen state and mode.
    - `m`: A pointer to a `mouse_event` structure containing details about the mouse event, such as button states and SGR type.
    - `x`: An unsigned integer representing the x-coordinate of the mouse event.
    - `y`: An unsigned integer representing the y-coordinate of the mouse event.
    - `rbuf`: A pointer to a constant character pointer where the resulting escape sequence buffer will be stored.
    - `rlen`: A pointer to a size_t variable where the length of the resulting escape sequence will be stored.
- **Control Flow**:
    - Initialize the output buffer and length to NULL and 0, respectively.
    - Check if the screen mode allows processing of motion events; if not, return 0 to discard the event.
    - Check if the event is a release event and if the screen mode allows it; if not, return 0 to discard the event.
    - Determine the appropriate escape sequence format based on the screen mode and mouse event details (SGR, UTF-8, or legacy format).
    - If the SGR format is used, construct the escape sequence using the SGR details from the mouse event.
    - If the UTF-8 format is used, ensure the parameters are within valid range and construct the escape sequence using UTF-8 encoding.
    - If the legacy format is used, ensure the parameters are within valid range and construct the escape sequence using the legacy encoding.
    - Store the constructed escape sequence and its length in the provided pointers `rbuf` and `rlen`.
    - Return 1 to indicate successful processing of the mouse event.
- **Output**:
    - The function returns an integer indicating success (1) or failure (0) in processing the mouse event, and outputs the constructed escape sequence and its length through the `rbuf` and `rlen` pointers.
- **Functions called**:
    - [`input_key_split2`](#input_key_split2)


---
### input_key_mouse<!-- {{#callable:input_key_mouse}} -->
The `input_key_mouse` function processes mouse events for a specific window pane, translating them into a format suitable for the application running in the pane.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure representing the pane where the mouse event occurred.
    - `m`: A pointer to a `mouse_event` structure containing details about the mouse event to be processed.
- **Control Flow**:
    - Check if the mouse event should be ignored or if the pane is not in a mouse mode; if so, return immediately.
    - Determine the mouse event's position within the pane using `cmd_mouse_at`; if unsuccessful, return.
    - Check if the pane is visible using `window_pane_visible`; if not, return.
    - Attempt to get the mouse event string using [`input_key_get_mouse`](#input_key_get_mouse); if unsuccessful, return.
    - Log the mouse event data for debugging purposes.
    - Write the mouse event data to the pane's event buffer using [`input_key_write`](#input_key_write).
- **Output**:
    - The function does not return a value; it performs actions based on the mouse event and writes data to the pane's event buffer.
- **Functions called**:
    - [`input_key_get_mouse`](#input_key_get_mouse)
    - [`input_key_write`](#input_key_write)


# Function Declarations (Public API)

---
### input_key_mouse<!-- {{#callable_declaration:input_key_mouse}} -->
Processes a mouse event for a window pane.
- **Description**: This function handles mouse events for a specified window pane, translating them into a format suitable for the application running within the pane. It should be called when a mouse event occurs and the pane is visible and in a mouse mode. The function will ignore events if the mouse mode is not active or if the pane is not visible. It also checks the validity of the mouse event's position within the pane and processes it accordingly.
- **Inputs**:
    - `wp`: A pointer to the window pane structure where the mouse event occurred. Must not be null, and the pane should be visible and in a mouse mode for the function to process the event.
    - `m`: A pointer to the mouse event structure containing details of the mouse event. Must not be null, and the event should not be marked as ignored for processing to occur.
- **Output**: None
- **See also**: [`input_key_mouse`](#input_key_mouse)  (Implementation)


---
### input_key_cmp<!-- {{#callable_declaration:input_key_cmp}} -->
Compares two input key entries based on their key codes.
- **Description**: Use this function to compare two input key entries when sorting or searching within a data structure that organizes keys, such as a red-black tree. It returns an integer indicating the relative order of the two keys, which is useful for maintaining sorted order or for equality checks. This function assumes that both input parameters are valid pointers to `input_key_entry` structures and does not handle null pointers.
- **Inputs**:
    - `ike1`: A pointer to the first `input_key_entry` structure to compare. Must not be null.
    - `ike2`: A pointer to the second `input_key_entry` structure to compare. Must not be null.
- **Output**: Returns -1 if the first key is less than the second, 1 if greater, and 0 if they are equal.
- **See also**: [`input_key_cmp`](#input_key_cmp)  (Implementation)


